---
title: needed-bits optimizations in guile — wingolog
url: https://wingolog.org/archives/2024/09/26/needed-bits-optimizations-in-guile
published: "2024-09-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/09/26/needed-bits-optimizations-in-guile
---

# needed-bits optimizations in guile — wingolog

## [needed-bits optimizations in guile](/archives/2024/09/26/needed-bits-optimizations-in-guile)

26 September 2024 12:44 PM

- [guile](/tags/guile)
- [flow analysis](/tags/flow%20analysis)
- [cps soup](/tags/cps%20soup)
- [needed-bits](/tags/needed-bits)
- [unboxing](/tags/unboxing)
- [fixnums](/tags/fixnums)
- [optimizations](/tags/optimizations)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)

Hey all, I had a fun bug this week and want to share it with you.

### numbers and representations

First, though, some background. Guile’s numeric operations are defined over the complex numbers, not over e.g. a finite field of integers. This is generally great when writing an algorithm, because you don’t have to think about how the computer will actually represent the numbers you are working on.

In practice, Guile will represent a small exact integer as a [*fixnum*](https://www.gnu.org/software/guile/manual/html_node/Faster-Integers.html), which is a machine word with a low-bit tag. If an integer doesn’t fit in a word (minus space for the tag), it is represented as a heap-allocated bignum. But sometimes the compiler can realize that e.g. the operands to a specific bitwise-and operation are within (say) the 64-bit range of unsigned integers, and so therefore we can use *unboxed operations* instead of the more generic functions that do run-time dispatch on the operand types, and which might perform heap allocation.

Unboxing is important for speed. It’s also tricky: under what circumstances can we do it? In the example above, there is information that flows from defs to uses: the operands of `logand` are known to be exact integers in a certain range and the operation itself is closed over its domain, so we can unbox.

But there is another case in which we can unbox, in which information flows backwards, from uses to defs: if we see `(logand n #xff)`, we know:

- the result will be in `[0, 255]`

- that *n* will be an exact integer (or an exception will be thrown)

- we are only interested in a subset of *n*‘s bits.

Together, these observations let us transform the more general `logand` to an unboxed operation, having first truncated *n* to a `u64`. And actually, the information can flow from use to def: if we know that *n* will be an exact integer but don’t know its range, we can transform the potentially heap-allocating computation that produces *n* to instead truncate its result to the `u64` range where it is defined, instead of just truncating at the use; and potentially this information could travel farther up the dominator tree, to inputs of the operation that defines *n*, their inputs, and so on.

### needed-bits: the `|0` of scheme

Let’s say we have a numerical operation that produces an exact integer, but we don’t know the range. We could truncate the result to a `u64` and use unboxed operations, if and only if only `u64` bits are used. So we need to compute, for each variable in a program, what bits are needed from it.

I think this is generally known a *needed-bits analysis*, though both Google and my textbooks are failing me at the moment; perhaps this is because dynamic languages and flow analysis don’t get so much attention these days. Anyway, the analysis can be local (within a basic block), global (all blocks in a function), or interprocedural (larger than a function). Guile’s is global. Each CPS/SSA variable in the function starts as needing 0 bits. We then compute the fixpoint of visiting each term in the function; if a term causes a variable to flow out of the function, for example via return or call, the variable is recorded as needing all bits, as is also the case if the variable is an operand to some primcall that doesn’t have a specific needed-bits analyser.

Currently, only `logand` has a needed-bits analyser, and this is because sometimes you want to do modular arithmetic, for example in a hash function. Consider Bon Jenkins’ [lookup3 string hash
function](http://burtleburtle.net/bob/c/lookup3.c):

```
#define rot(x,k) (((x)<<(k)) | ((x)>>(32-(k))))
#define mix(a,b,c) \
{ \
  a -= c;  a ^= rot(c, 4);  c += b; \
  b -= a;  b ^= rot(a, 6);  a += c; \
  c -= b;  c ^= rot(b, 8);  b += a; \
  a -= c;  a ^= rot(c,16);  c += b; \
  b -= a;  b ^= rot(a,19);  a += c; \
  c -= b;  c ^= rot(b, 4);  b += a; \
}
...

```

If we [transcribe this to
Scheme](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/guile-vm.scm#n37), we get something like:

```
(define (jenkins-lookup3-hashword2 str)
  (define (u32 x) (logand x #xffffFFFF))
  (define (shl x n) (u32 (ash x n)))
  (define (shr x n) (ash x (- n)))
  (define (rot x n) (logior (shl x n) (shr x (- 32 n))))
  (define (add x y) (u32 (+ x y)))
  (define (sub x y) (u32 (- x y)))
  (define (xor x y) (logxor x y))

  (define (mix a b c)
    (let* ((a (sub a c)) (a (xor a (rot c 4)))  (c (add c b))
           (b (sub b a)) (b (xor b (rot a 6)))  (a (add a c))
           (c (sub c b)) (c (xor c (rot b 8)))  (b (add b a))
           ...)
      ...))

  ...

```

These `u32` calls are like [the JavaScript `|0`
idiom](https://stackoverflow.com/questions/30156122/what-is-this-asm-style-x-0-some-javascript-programmers-are-now-using), to tell the compiler that we really just want the low 32 bits of the number, as an integer. Guile’s compiler will propagate that information down to uses of the defined values but also back up the dominator tree, resulting in unboxed arithmetic for all of these operations.

(When writing this, I got all the way here and then realized [I had
already written quite a bit about this, almost a decade ago
ago](https://wingolog.org/archives/2016/01/19/unboxing-in-guile). Oh well, consider this your lucky day, you get two scoops of prose!)

### the bug

All that was just prelude. So I said that needed-bits is a fixed-point flow analysis problem. In this case, I want to compute, for each variable, what bits are needed for its definition. Because of loops, we need to keep iterating until we have found the fixed point. We use a worklist to represent the conts we need to visit.

Visiting a cont may cause the program to require more bits from the variables that cont uses. [Consider](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/specialize-numbers.scm#n306):

```
(define-significant-bits-handler
    ((logand/immediate label types out res) param a)
  (let ((sigbits (sigbits-intersect
                   (inferred-sigbits types label a)
                   param
                   (sigbits-ref out res))))
    (intmap-add out a sigbits sigbits-union)))

```

This is the sigbits (needed-bits) handler for `logand` when one of its operands ( *param*) is a constant and the other ( *a*) is variable. It adds an entry for *a* to the analysis *out*, which is an intmap from variable to a bitmask of needed bits, or `#f` for all bits. If *a* already has some computed sigbits, we add to that set via `sigbits-union`. The interesting point comes in the `sigbits-intersect` call: the bits that we will need from *a* are first the bits that we infer *a* to have, by forward type-and-range analysis; intersected with the bits from the immediate *param*; intersected with the needed bits from the result value *res*.

If the `intmap-add` call is idempotent—i.e., *out* already contains *sigbits* for *a*—then *out* is returned as-is. So we can check for a fixed-point by comparing *out* with the resulting analysis, via `eq?`. If they are not equal, we need to add the cont that defines *a* to the worklist.

The bug? The bug was that we were not enqueuing the def of *a*, but rather the predecessors of *label*. This works when there are no cycles, provided we visit the worklist in post-order; and regardless, it works for many other analyses in Guile where we compute, for each labelled cont (basic block), [some set of facts about all other labels
or about all other
variables](https://wingolog.org/archives/2014/07/01/flow-analysis-in-guile). In that case, enqueuing a predecessor on the worklist will cause all nodes up and to including the variable’s definition to be visited, because each step adds more information (relative to the analysis computed on the previous visit). But it doesn’t work for this case, because we aren’t computing a per-label analysis.

The solution was to [rewrite that particular fixed-point to enqueue
labels that define a variable (possibly multiple defs, because of joins
and loop back-edges), instead of just the predecessors of the use](https://git.savannah.gnu.org/cgit/guile.git/commit/?id=aff9ac968840e9c86719fb613bd2ed3c39b9905c).

Et voilà ! If you got this far, bravo. Type at y’all again soon!

## related articles

- [approaching cps soup](/archives/2023/05/20/approaching-cps-soup)
- [unboxing in guile](/archives/2016/01/19/unboxing-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [flow analysis in guile](/archives/2014/07/01/flow-analysis-in-guile)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)

### 2 responses

1. [Cassie Jones](https://www.witchoflight.com) says:[1 October 2024 9:04 PM](#b0e42443369faa26b4c5b92839d7cd5aa0eacc1b)

   I think one reason you’re not seeing many results for “needed bits” is because things like LLVM at least call this analysis “demanded bits”. Thanks for the article! I love to see nice dataflows solving problems.

2. [soap2day](https://www.soap2dey.com/action/) says:[2 October 2024 11:11 AM](#8067260a9c3920be9ecc4d7e18f89a72856fcc4d)

   I love to see nice dataflows solving problems.

Comments are closed.
