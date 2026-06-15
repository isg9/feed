---
title: instruction explosion in guile — wingolog
url: https://wingolog.org/archives/2018/01/17/instruction-explosion-in-guile
published: "2018-01-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/01/17/instruction-explosion-in-guile
---

# instruction explosion in guile — wingolog

## [instruction explosion in guile](/archives/2018/01/17/instruction-explosion-in-guile)

17 January 2018 10:30 AM

- [guile](/tags/guile)
- [compilers](/tags/compilers)
- [instruction explosion](/tags/instruction%20explosion)
- [loop optimizations](/tags/loop%20optimizations)
- [cps](/tags/cps)

Greetings, fellow Schemers and compiler nerds: I bring fresh nargery!

**instruction explosion**

A couple years ago I made a list of [compiler tasks for Guile](https://wingolog.org/archives/2016/02/04/guile-compiler-tasks). Most of these are still open, but I've been chipping away at the one labeled "instruction explosion":

> Now we get more to the compiler side of things. Currently in Guile's VM there are instructions like [vector-ref](https://www.gnu.org/software/guile/docs/master/guile.html/Inlined-Scheme-Instructions.html#Inlined-Scheme-Instructions). This is a little silly: there are also instructions to branch on the type of an object ( [br-if-tc7](https://www.gnu.org/software/guile/docs/master/guile.html/Branch-Instructions.html#Branch-Instructions) in this case), to get the vector's length, and to do a branching integer comparison. Really we should replace vector-ref with a combination of these test-and-branches, with real control flow in the function, and then the actual ref should use some more primitive unchecked memory reference instruction. Optimization could end up hoisting everything but the primitive unchecked memory reference, while preserving safety, which would be a win. But probably in most cases optimization wouldn't manage to do this, which would be a lose overall because you have more instruction dispatch.
>
> Well, this transformation is something we need for native compilation anyway. I would accept a patch to do this kind of transformation on the master branch, after version 2.2.0 has forked. In theory this would remove most all high level instructions from the VM, making the bytecode closer to a virtual CPU, and likewise making it easier for the compiler to emit native code as it's working at a lower level.

Now that I'm getting close to finished I wanted to share some thoughts. [Previous progress reports on the mailing list](https://lists.gnu.org/archive/html/guile-devel/2018-01/msg00003.html).

**a simple loop**

As an example, consider this loop that sums the 32-bit floats in a bytevector. I've annotated the code with lines and columns so that you can correspond different pieces to the assembly.

```
   0       8   12     19
 +-v-------v---v------v-
 |
1| (use-modules (rnrs bytevectors))
2| (define (f32v-sum bv)
3|   (let lp ((n 0) (sum 0.0))
4|     (if (< n (bytevector-length bv))
5|         (lp (+ n 4)
6|             (+ sum (bytevector-ieee-single-native-ref bv n)))
7|          sum)))

```

The assembly for the loop before instruction explosion went like this:

```
L1:
  17    (handle-interrupts)     at (unknown file):5:12
  18    (uadd/immediate 0 1 4)
  19    (bv-f32-ref 1 3 1)      at (unknown file):6:19
  20    (fadd 2 2 1)            at (unknown file):6:12
  21    (s64 L4
  23    (mov 1 0)               at (unknown file):5:8
  24    (j -7)                 ;; -> L1

```

So, already Guile's compiler has hoisted the `(bytevector-length bv)` and unboxed the loop index *n* and accumulator *sum*. This work aims to simplify further by exploding `bv-f32-ref`.

**exploding the loop**

In practice, instruction explosion happens in CPS conversion, as we are converting the Scheme-like [Tree-IL](https://www.gnu.org/software/guile/docs/master/guile.html/Tree_002dIL.html#Tree_002dIL) language down to the [CPS soup](https://www.gnu.org/software/guile/docs/master/guile.html/Continuation_002dPassing-Style.html#Continuation_002dPassing-Style) language. When we see a Tree-Il primcall (a call to a known primitive), instead of lowering it to a corresponding CPS primcall, we inline a whole blob of code.

In the concrete case of `bv-f32-ref`, we'd inline it with something like the following:

```
(unless (and (heap-object? bv)
             (eq? (heap-type-tag bv) %bytevector-tag))
  (error "not a bytevector" bv))
(define len (word-ref bv 1))
(define ptr (word-ref bv 2))
(unless (and (<= 4 len)
             (<= idx (- len 4)))
  (error "out of range" idx))
(f32-ref ptr len)

```

As you can see, there are four branches hidden in the `bv-f32-ref`: two to check that the object is a bytevector, and two to check that the index is within range. In this explanation we assume that the offset *idx* is already unboxed, but actually unboxing the index ends up being part of this work as well.

One of the goals of instruction explosion was that by breaking the operation into a number of smaller, more orthogonal parts, native code generation would be easier, because the compiler would only have to know about those small bits. However without an optimizing compiler, it would be better to [reify a call out to a specialized `bv-f32-ref` runtime routine](https://docs.google.com/document/d/15hmBrCrmMZzra8ekhl7meQZBodeahsauilNDImif5Xg/edit#heading=h.8zevmmfemvcv) instead of inlining all of this code -- probably whatever language you write your runtime routine in (C, rust, whatever) will do a better job optimizing than your compiler will.

But with an optimizing compiler, there is the possibility of removing possibly everything but the `f32-ref`. Guile doesn't quite get there, but almost; here's the post-explosion optimized assembly of the inner loop of f32v-sum:

```
L1:
  27    (handle-interrupts)
  28    (tag-fixnum 1 2)
  29    (s64 L5
  31    (uadd/immediate 0 2 4)  at (unknown file):5:12
  32    (u64 L2
  34    (f32-ref 2 5 2)
  35    (fadd 3 3 2)            at (unknown file):6:12
  36    (mov 2 0)               at (unknown file):5:8
  37    (j -10)                ;; -> L1

```

**good things**

The first thing to note is that unlike the "before" code, there's no instruction in this loop that can throw an exception. Neat.

Next, note that there's no type check on the bytevector; the [peeled iteration](https://wingolog.org/archives/2015/07/28/loop-optimizations-in-guile) preceding the loop already proved that the bytevector is a bytevector.

And indeed there's no reference to the bytevector at all in the loop! The value being dereferenced in `(f32-ref 2 5 2)` is a raw pointer. (Read this instruction as, "sp\[2\] = \*(float\*)((byte\*)sp\[5\] + (uptrdiff\_t)sp\[2\])".) The compiler does something interesting; the `f32-ref` CPS primcall actually takes three arguments: the garbage-collected object protecting the pointer, the pointer itself, and the offset. The object itself doesn't appear in the residual code, but including it in the `f32-ref` primcall's inputs keeps it alive as long as the `f32-ref` itself is alive.

**bad things**

Then there are the limitations. Firstly, instruction 28 tags the u64 loop index as a fixnum, but never uses the result. Why is this here? Sadly it's because the value is used in the bailout at L2. Recall this pseudocode:

```
(unless (and (<= 4 len)
             (<= idx (- len 4)))
  (error "out of range" idx))

```

Here the error ends up lowering to a [`throw` CPS term](https://lists.gnu.org/archive/html/guile-devel/2018-01/msg00003.html) that the compiler recognizes as a bailout and renders out-of-line; cool. But it uses *idx* as an argument, as a tagged SCM value. The compiler untags the loop index, but has to keep a tagged version around for the error cases.

The right fix is probably some kind of allocation sinking pass that sinks the `tag-fixnum` to the bailouts. Oh well.

Additionally, there are two tests in the loop. Are both necessary? Turns out, yes :( Imagine you have a bytevector of length 102 **5**. The loop continues until the last ref at offset 1024, which is within bounds of the bytevector but there's one one byte available at that point, so we need to throw an exception at this point. The compiler did as good a job as we could expect it to do.

**is is worth it? where to now?**

On the one hand, instruction explosion is a step sideways. The code is more optimal, but it's more instructions. Because Guile currently has a bytecode VM, that means more total interpreter overhead. Testing on a 40-megabyte bytevector of 32-bit floats, the exploded `f32v-sum` completes in 115 milliseconds compared to around 97 for the earlier version.

On the other hand, it is very easy to imagine how to compile these instructions to native code, either ahead-of-time or via a simple template JIT. You practically just have to look up the instructions in the corresponding ISA reference, is all. The result should perform quite well.

I will probably take a whack at a simple template JIT first that does no register allocation, then ahead-of-time compilation with register allocation. Getting the AOT-compiled artifacts to dynamically link with runtime routines is a sufficient pain in my mind that I will put it off a bit until later. I also need to figure out a good strategy for truly polymorphic operations like general integer addition; probably involving inline caches.

So that's where we're at :) Thanks for reading, and happy hacking in Guile in 2018!

## related articles

- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [approaching cps soup](/archives/2023/05/20/approaching-cps-soup)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [cps soup](/archives/2015/07/27/cps-soup)
- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)

### 5 responses

1. non-e-moose says:[17 January 2018 7:37 PM](#471fd3941d38c0eacee7efce4359b6d74a2d0f58)

   Another example of an inept developer talking about "instructions" when their only understanding is that of an IR (Itermediate Representation)

   "I also need to figure out a good strategy for truly polymorphic operations like general integer addition;"

   A great start would be to understand actual instruction sequences.

   If you are (and you seem to be) pick clang as a way to learn about code sequences.

2. [John Cowan](http://vrici.lojban.org/~cowan) says:[18 January 2018 1:11 AM](#35ac5200d35f42a7c5bd2ef00c1a5ba88b6bfda6)

   Everybody's understanding is only at the level of an intermediate representation unless they know exactly how the CPU works at the circuit level.

3. Stephen Eilert says:[18 January 2018 1:20 AM](#1b269dfc4493f652b6fdd9eb47458f27f8f32662)

   "Another example of an inept developer talking about "instructions" when their only understanding is that of an IR "

   How interesting would a world be where Andy Wingo would be considered an "inept" developer. I wonder which category would everyone else fall into.

4. rixed says:[19 January 2018 6:24 AM](#6336616a76757dee7bdcfa4e027418c7d89f0ba4)

   "Another example of an inept developer"

   Could you comment on the ideas rather than the person?

   I guess we'd all like to learn something about what's so special about instruction sequences that made you dismiss the whole post like this. If your comment was anything else than an attempt at AI, that is.

5. [Arne Babenhauserheide](http://draketo.de) says:[21 January 2018 9:17 PM](#90a5515940add5a9e154ac0e116b42ef4c1811a1)

   …when they start to bite you unfairly, you know you are getting the attention from those who consider themselves to be above you…

Comments are closed.
