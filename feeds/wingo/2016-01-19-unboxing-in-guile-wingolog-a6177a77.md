---
title: unboxing in guile — wingolog
url: https://wingolog.org/archives/2016/01/19/unboxing-in-guile
published: "2016-01-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/01/19/unboxing-in-guile
---

# unboxing in guile — wingolog

## [unboxing in guile](/archives/2016/01/19/unboxing-in-guile)

19 January 2016 11:57 AM

- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [scalar replacement](/tags/scalar%20replacement)
- [unboxing](/tags/unboxing)
- [common subexpression elimination](/tags/common%20subexpression%20elimination)
- [type inference](/tags/type%20inference)
- [flow analysis](/tags/flow%20analysis)

Happy snowy Tuesday, hackfolk! I know I said in my last dispatch that I'd write about Lua soon, but that article is still cooking. In the meantime, a note on Guile and unboxing.

**on boxen, on blitzen**

Boxing is a way for a programming language implementation to represent a value.

A boxed value is the combination of a value along with a tag providing some information about the value. Both the value and the tag take up some space. The value can be thought to be inside a "box" labelled with the tag and containing the value.

A value's tag can indicate whether the value's bits should be interpreted as an unsigned integer, as a double-precision floating-point number, as an array of words of a particular data type, and so on. A tag can also be used for other purposes, for example to indicate whether a value is a pointer or an "immediate" bit string.

Whether values in a programming language are boxed or not is an implementation consideration. It can be the case that in languages with powerful type systems that a compiler can know what the representation of all values are in all parts of all programs, and so boxing is never needed. However, it's much easier to write a garbage collector if values have a somewhat uniform representation, with tag bits to tell the GC how to trace any pointers that might be contained in the object. Tags can also carry run-time type information needed by a dynamically typed language like Scheme or JavaScript, to allow for polymorphic predicates like `number?` or `pair?`.

Boxing all of the values in a program can incur significant overhead in space and in time. For example, one way to implement boxes is to allocate space for the tag and the value on the garbage-collected heap. A boxed value would then be referred to via a pointer to the corresponding heap allocation. However, most memory allocation systems align their heap allocations on word-sized boundaries, for example on 8-byte boundaries. That means that the low 3 bits of a heap allocation will always be zero. If you make a bit string whose low 3 bits are not zero, it cannot possibly be a valid pointer. In that case you can represent some types within the set of bit strings that cannot be valid pointers. These values are called "immediates", as opposed to "heap objects". [In Guile, we have immediate representations for characters, booleans, some special values, and a subset of the integers.](http://git.savannah.gnu.org/cgit/guile.git/tree/libguile/tags.h#n130) [Alternately, a programming language implementation can represent values as double-precision floating point numbers, and shove pointers into the space of the NaN values.](https://wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations) And for heap allocations, [some systems can associate one tag with a whole page of values, minimizing per-value boxing overhead](https://www.cs.indiana.edu/~dyb/pubs/bibop.pdf).

The goal of these optimizations is to avoid heap allocation for some kinds of boxes. While most language implementations have good garbage collectors that make allocation fairly cheap, the best way to minimize allocation cost is to refrain from it entirely.

In Guile's case, we currently use a combination of low-bit tagging for immediates, including fixnums (a subset of the integers), and tagged boxes on the heap for everything else, including floating-point numbers.

Boxing floating-point numbers obviously incurs huge overhead on floating-point math. You have to consider that each intermediate value produced by a computation will result in the allocation of another 8 bytes for the value and 4 or 8 bytes for the tag. Given that Guile aligns allocations on 8-byte boundaries, the result is a 16-byte allocation in either case. Consider this loop to sum the doubles in a bytevector:

```
(use-modules (rnrs bytevectors))
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (+ sum (bytevector-ieee-double-native-ref v i)))
        sum)))

```

Each trip through the loop is going to allocate not one but two heap floats: one to box the result of `bytevector-ieee-double-native-ref` (whew, what a mouthful), and one for the sum. If we have a bytevector of 10 million elements, that will be 320 megabytes of allocation. Guile can allocate short-lived 16-byte allocations at about 900 MB/s on my machine, so summing this vector is going to take at least 350ms, just for the allocation. Indeed, without unboxing I measure this loop at 580ms for a 10 million element vector:

```
> (define v (make-f64vector #e10e6 1.0))
> ,time (f64-sum v)
$1 = 1.0e7
;; 0.580114s real time, 0.764572s run time.  0.268305s spent in GC.

```

The run time is higher than the real time due to parallel marking. I think in this case, allocation has even higher overhead because it happens outside the bytecode interpreter. The `add` opcode has a fast path for small integers (fixnums), and if it needs to work on flonums it calls out to a C helper. That C helper doesn't have a pointer to the thread-local freelist so it has to go through a more expensive allocation path.

Anyway, in the time that Guile takes to fetch one f64 value from the vector and add it to the sum, the CPU ticked through some 150 cycles, so surely we can do better than this.

**unboxen, unblitzen**

Let's take a look again at the loop to see where the floating-point allocations are produced.

```
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (+ sum (bytevector-ieee-double-native-ref v i)))
        sum)))

```

It turns out there's no reason for the loquatiously-named `bytevector-ieee-double-native-ref` to return a boxed number. It's a monomorphic function that is well-known to the Guile compiler and virtual machine, and it even has its own opcode. In Guile 2.0 and until just a couple months ago in Guile 2.2, this function did box its return value, but that was because the virtual machine had no facility for unboxed values of any kind.

To allow `bytevector-ieee-double-native-ref` to return an unboxed double value, the first item of business was then to support unboxed values in Guile's VM. Looking forward to unboxed doubles, we made a change such that all on-stack values are 64 bits wide, even on 32-bit systems. (For simplicity, all locals in Guile take up the same amount of space. For the same reason, fetching 32-bit floats also unbox to 64-bit doubles.)

We also made a change to Guile's "stack maps", which are data structures that tell the garbage collector which locals are live in a stack frame. There is a stack map recorded at every call in a procedure, to be used when an activation is pending on the stack. Stack maps are stored in a side table in a separate section of the compiled ELF library. Live values are traced by the garbage collector, and dead values are replaced by a special "undefined" singleton. The change we made was to be able to indicate that live values were boxed or not, and if they were unboxed, what type they were (e.g. unboxed double). Knowing the type of locals helps the debugger to print values correctly. Currently, all unboxed values are immediates, so the GC doesn't need to trace them, but it's conceivable that we could have unboxed pointers at some point. Anyway, instead of just storing one bit (live or dead) per local in the stack map, we store two, and reserve one of the bit patterns to indicate that

the local is actually an f64 value.

But the changes weren't done then: since we had never had unboxed locals, there were quite a few debugging-related parts of the VM that assumed that we could access the first slot in an activation to see if it was a procedure. This dated from a time in Guile where slot 0 would always be the procedure being called, but the check is bogus ever since Guile 2.2 allowed local value slots corresponding to the closure or procedure arguments to be re-used for other values, if the closure or argument was dead. Another nail in the coffin of procedure-in-slot-0 was driven by closure optimizations, in which closures whose callees are all visible could specialize the representation of their closure in non-standard ways. It took a while, but unboxing f64 values flushed out these bogus uses of slot 0.

The next step was to add boxing and unboxing operations to the VM ( `f64->scm` and `scm->f64`, respectively). Then we changed `bytevector-ieee-double-native-ref` to return an unboxed value and then immediately box it via `f64->scm`. Similarly for `bytevector-ieee-double-native-set!`, we unbox the value via `scm->f64`, potentially throwing a type error. Unfortunately our run-time type mismatch errors got worse; although the source location remains the same, `scm->f64` doesn't include the reason for the unboxing. Oh well.

```
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (let ((f64 (bytevector-ieee-double-native-ref v i))
                  (boxed (f64->scm f64)))
              (+ sum boxed))
        sum)))

```

When we lower Tree-IL to CPS, we insert the needed `f64->scm` and `scm->f64` boxing and unboxing operations around bytevector accesses. Cool. At this point we have a system with unboxed f64 values, but which is slower than the original version because every f64 bytevector access involves two instructions instead of one, although the instructions themselves together did the same amount of work. However, telling the optimizer about these instructions could potentially eliminate some of them. Let's keep going and see where we get.

Let's attack the other source of boxes, the accumulation of the sum. We added some specialized instuctions to the virtual machine to support arithmetic over unboxed values. Doing this is potentially a huge win, because not only do you avoid allocating a box for the result, you also avoid the type checks on the incoming values. So we add `f64+`, `f64-`, and so on.

Unboxing the `+` to `f64+` is a tricky transformation, and relies on [type analysis](https://wingolog.org/archives/2015/10/29/type-folding-in-guile). Our assumption is that if type analysis indicates that we are in fact able to replace a generic arithmetic instruction with a combination of operand unboxing, unboxed arithmetic, and a boxing operation, then we should do it. Separating out the boxes and the monomorphic arithmetic opens the possibility to remove the resulting box, and possibly remove the unboxing of operands too. In this case, we run an optimization pass and end up with something like:

```
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (let ((f64 (bytevector-ieee-double-native-ref v i))
                  (boxed (f64->scm f64)))
              (f64->scm
               (f64+ (scm->f64 sum)
                     (scm->f64 boxed)))))
        sum)))

```

[Scalar replacement via fabricated expressions](https://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile) will take the definition of `boxed` as `(f64->scm f64)` and fabricate a definition of `f64` as `(scm->f64 boxed)`, which propagates down to the `f64+` so we get:

```
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (let ((f64 (bytevector-ieee-double-native-ref v i))
                  (boxed (f64->scm f64)))
              (f64->scm
               (f64+ (scm->f64 sum)
                     f64))))
        sum)))

```

Dead code elimination can now kill `boxed`, so we end up with:

```
(define (f64-sum v)
  (let lp ((i 0) (sum 0.0))
    (if (< i (bytevector-length v))
        (lp (+ i 8)
            (let ((f64 (bytevector-ieee-double-native-ref v i)))
              (f64->scm
               (f64+ (scm->f64 sum)
                     f64))))
        sum)))

```

Voilà, we removed one allocation. Yay!

As we can see from the residual code, we're still left with one `f64->scm` boxing operation. That expression is one of the definitions of `sum`, one of the loop variables. The other definition is `0.0`, the starting value. So, after specializing arithmetic operations, we go through the set of multiply-defined variables ("phi" variables) and see what we can do to unbox them.

A phi variable can be unboxed if all of its definitions are unboxable. It's not always clear that you *should* unbox, though. For example, maybe you know via looking at the definitions for the value that it can be unboxed as an f64, but all of its uses are boxed. In that case it could be that you throw away the box when unboxing each definition, only to have to re-create them anew when using the variable. You end up allocating twice as much instead of not at all. It's a tricky situation. Currently we assume a variable with multiple definitions should only be unboxed if it has an unboxed use. The initial set of unboxed uses is the set of operands to `scm->f64`. We iterate this set to a fixed point: unboxing one phi variable could cause others to be unbox as well. As a heuristic, we only require one unboxed use; it could be there are other uses that are boxed, and we could indeed hit that pessimal double-allocation case. Oh well!

In this case, the intermediate result looks something like:

```
(define (f64-sum v)
  (let lp ((i 0) (sum (scm->f64 0.0)))
    (let ((sum-box (f64->scm sum)))
      (if (< i (bytevector-length v))
          (lp (+ i 8)
              (let ((f64 (bytevector-ieee-double-native-ref v i)))
                (scm->f64
                 (f64->scm
                  (f64+ (scm->f64 sum-box)
                        f64))))
          sum-box)))

```

After the scalar replacement and dead code elimination passes, we end up with something more like:

```
(define (f64-sum v)
  (let lp ((i 0) (sum (scm->f64 0.0)))
    (let ((sum-box (f64->scm sum)))
      (if (< i (bytevector-length v))
          (lp (+ i 8)
              (f64+ sum
                    (bytevector-ieee-double-native-ref v i)))
          sum-box)))

```

Well this is looking pretty good. There's still a box though. Really we should sink this to the exit, but as it happens there's something else that accidentally works in our favor: [loop peeling](https://wingolog.org/archives/2015/07/28/loop-optimizations-in-guile). By peeling the first loop iteration, we create a control-flow join at the loop exit that defines a phi variable. That phi variable is subject to the same optimization, sinking the box down to the join itself. So in reality the result looks like:

```
(define (f64-sum v)
  (let ((i 0)
        (sum (scm->f64 0.0))
        (len (bytevector-length v)))
    (f64->scm
     (if (< i len)
         sum
         (let ((i (+ i 8))
               (sum (f64+ sum
                          (bytevector-ieee-double-native-ref v i))))
           (let lp ((i i) (sum sum))
             (if (< i len)
                 (lp (+ i 8)
                     (f64+ sum (bytevector-ieee-double-native-ref v i)))
                 sum)))))))

```

As you can see, the peeling lifted the length computation up to the top too, which is a bonus. We should probably still implement allocation sinking, especially for loops for which peeling isn't an option, but the current status often works well. Running `f64-sum` on a 10-million-element packed double array goes down from 580ms to 99ms, or to some 25 or 30 CPU cycles per element, and of course no time in GC. Considering that this loop still has the overhead of bytecode interpretation and cache misses, I think we're doing A O K.

**limits**

It used to be that using packed bytevectors of doubles was an easy way to make your program slower using types (thanks to Sam Tobin-Hochstadt for that quip). The reason is that although a packed vector of doubles uses less memory, every access to it has to allocate a new boxed number. Compare to "normal" vectors where sure, it uses more memory, but fetching an element fetches an already-boxed value. Now with the unboxing optimization, this situation is properly corrected... in most cases.

The major caveat is that for unboxing to work completely, each use of a potentially-unboxable value has to have an alternate implementation that can work on unboxed values. In our example above, the only use was `f64+` (which internally is really called `fadd`), so we win. Writing an f64 to a bytevector can also be unboxed. Unfortunately, bytevectors and simple arithmetic are currently all of the unboxable operations. We'll implement more over time, but it's a current limitation.

Another point is that we are leaning heavily on the optimizer to remove the boxes when it can. If there's a bug or a limitation in the optimizer, it could be the box stays around needlessly. It happens, hopefully less and less but it does happen. To be sure you get the advantages, you need to time the code and see if it's spending significant time in GC. If it is, then you need to disassemble your code to see where that's happening. It's not a very nice thing, currently. The Scheme-like representations I gave above were written by hand; the CPS intermediate language is much more verbose than that.

Another limitation is that function arguments and return values are always boxed. Of course, the compiler can inline and contify a lot of functions, but that means that to use abstraction, you need to build up a mental model of what the inliner is going to do.

Finally, it's not always obvious to the compiler what the type of a value is, and that necessarily limits unboxing. For example, if we had started off the loop by defining `sum` to be `0` instead of `0.0`, the result of the loop as a whole could be either an exact integer or an inexact real. Of course, loop peeling mitigates this to an extent, unboxing `sum` within the loop after the first iteration, but it so happens that peeling also prevents the phi join at the loop exit from being unboxed, because the result from the peeled iteration is `0` and not `0.0`. In the end, we are unable to remove the equivalent of `sum-box`, and so we still allocate once per iteration. Here is a clear case where we would indeed need allocation sinking.

Also, consider that in other contexts the type of `(+ x 1.0)` might actually be complex instead of real, which means that depending on the type of *x* it might not be valid to unbox this addition. Proving that a number is not complex can be non-obvious. That's the second way that fetching a value from a packed vector of doubles or floats is useful: it's one of the rare times that you know that a number is real-valued.

**on integer, on fixnum**

That's all there is to say about floats. However, when doing some benchmarks of the floating-point unboxing, one user couldn't reproduce some of the results: they were seeing huge run-times for on a microbenchmark that repeatedly summed the elements of a vector. It turned out that the reason was that they were on a 32-bit machine, and one of the loop variables used in the test was exceeding the fixnum range. Recall that fixnums are the subset of integers that fit in an immediate value, along with their tag. Guile's fixnum tag is 2 bits, and fixnums have a sign bit, so the most positive fixnum on a 32-bit machine is 229—1, or around 500 million. It sure is a shame not to be able to count up to `#xFFFFFFFF` without throwing an allocation party!

So, we set about seeing if we could unbox integers as well in Guile. Guile's compiler has a lot more visibility as to when something is an integer, compared to real numbers. Anything used as an index into a vector or similar data structure must be an exact integer, and any query as to the length of a vector or a string or whatever is also an integer.

Note that knowing that a value is an exact integer is insufficient to unbox it: you have to also know that it is within the range of your unboxed integer data type. Here we take advantage of the fact that [in Guile, type analysis also infers ranges](https://wingolog.org/archives/2015/10/29/type-folding-in-guile). So, cool. Because the kinds of integers that can be used as indexes and lengths are all non-negative, our first unboxed integer type is `u64`, the unsigned 64-bit integers.

If Guile did native compilation, it would always be a win to unbox any integer operation, if only because you would avoid polymorphism or any other potential side exit. For bignums that are within the unboxable range, the considerations are similar to the floating-point case: allocation costs dominate, so unboxing is almost always a win, provided that you avoid double-boxing. Eliminating one allocation can pay off a lot of instruction dispatch.

For fixnums, though, things are not so clear. Immediate tagging is such a cheap way of boxing that in an interpreter, the extra instructions you introduce could outweigh any speedup from having faster operations.

In the end, I didn't do science and I decided to just go ahead and unbox if I could. We are headed towards native compilation, this is a necessary step along that path, and what the hell, it seemed like a good idea at the time.

Because there are so many more integers in a typical program than floating-point numbers, we had to provide unboxed integer variants of quite a number of operations. Of course we could unconditionally require unboxed arguments to `vector-ref`, `string-length` and so on, but in addition to making u64 variants of arithmetic, we also support bit operations like `logand` and such. Unlike the current status with floating point numbers, we can do test-and-branch over unboxed u64 comparisons, and we can compare u64 values to boxed SCM values.

In JavaScript, making sure an integer is unboxed is easy: you just do `val | 0`. The bit operation `|` truncates the value to a uint32 32-bit two's-complement signed integer (thanks to [Slava](https://twitter.com/mraleph) for the correction). In Guile though, we have arbitrary-precision bit operations, so although `(logior val 0)` would assert that `val` is an integer, it wouldn't necessarily mean that it's unboxable.

Instead, the Guile idiom for making sure you have an unboxed integer in a particular range should go like this:

```
(define-inlinable (check-uint-range x mask)
  (let ((x* (logand x mask)))
    (unless (= x x*)
      (error "out of range" x))
    x*))

```

A helper like this is useful to assert that an argument to a function is of a particular type, especially given that arguments to functions are always boxed and treated as being of unknown type. The `logand` asserts that the value is an integer, and the comparison asserts that it is within range.

For example, if we want to implement a function that does modular 8-bit addition, it can go like:

```
(define-inlinable (check-uint8 x)
  (check-uint-range x #xff))
(define-inlinable (truncate-uint8 x)
  (logand x #xff))
(define (uint8+ x y)
  (truncate-uint8 (+ (check-uint8 x) (check-uint8 y))))

```

If we disassemble this function, we get something like:

```
Disassembly of # at #xa8d0f8:

   0    (assert-nargs-ee/locals 3 2)    ;; 5 slots (2 args)
   1    (scm->u64/truncate 4 3)
   2    (load-u64 1 0 255)
   5    (ulogand 4 4 1)
   6    (br-if-u64-=-scm 4 3 #f 17)     ;; -> L1
;; [elided code to throw an error if x is not in range]
L1:
  23    (scm->u64/truncate 3 2)
  24    (ulogand 3 3 1)
  25    (br-if-u64-=-scm 3 2 #f 18)     ;; -> L2
;; [elided code to throw an error if y is not in range]
L2:
  43    (uadd 4 4 3)
  44    (ulogand 4 4 1)
  45    (u64->scm 3 4)
  46    (return-values 2)               ;; 1 value

```

The `scm->u64/truncate` instructions unbox an integer, but truncating it to the u64 range. They are used when we know that any additional bits won't be used, as in this case where we immediately do a `logand` of the unboxed value. All in all it's not a bad code sequence; there are two possible side exits for each argument (not an integer signalled by the unboxing, and out of range signalled by the explicit check), and no other run-time dispatch. For now I think we can be pretty happy with the code.

That's about it for integer unboxing. We also support unboxed signed 64-bit integers, mostly for use as operands or return values from `bytevector-s8-ref` and similar unboxed accessors on bytevectors. There are fewer operations that have s64 variants, though, compared to u64 variants.

**summary**

Up until now in Guile, it could be that you might have to avoid Scheme if you needed to do some kinds of numeric computation. Unboxing floating-point and integer numbers makes it feasible to do more computation in Scheme instead of having to rely in inflexible C interfaces. At the same time, as a Scheme hacker I feel much more free knowing that I can work on 64-bit integers without necessarily allocating bignums. I expect this optimization to have a significant impact on the way I program, and what I program. We'll see where this goes, though. Until next time, happy hacking :)

## related articles

- [needed-bits optimizations in guile](/archives/2024/09/26/needed-bits-optimizations-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [flow analysis in guile](/archives/2014/07/01/flow-analysis-in-guile)
- [approaching cps soup](/archives/2023/05/20/approaching-cps-soup)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [lightening run-time code generation](/archives/2019/05/24/lightening-run-time-code-generation)

### 3 responses

1. [John Cowan](http://www.ccil.org/~cowan) says:[19 January 2016 7:40 PM](#b20cba385041e69e771796ccbed8ef445f917602)

   I'm a little bit concerned about automatically widening all values pulled out of f32 vectors to f64 without remembering that you've done so. Not only does that mean all float math is 64-bit (which may or may not be slower) but because extending f32s to f64s is inverse truncation rather than inverse rounding, you may get unexpected numerical values compared to Fortran or C single-precision math (technically C99 and up do not require this, they just permit it, but all non-toy compilers do it).

2. [Andy Wingo](https://wingolog.org/) says:[19 January 2016 8:25 PM](#13f02517e9b08bcc3ae561d650c3c18dc3dbbebf)

   For what it's worth, Guile always uses doubles for inexact real arithmetic, and has done so since before it was called Guile. Unboxing floats to doubles changes nothing in that regard.

3. [John Cowan](http://www.ccil.org/~cowan) says:[19 January 2016 11:10 PM](#51ec96157ade4d326aca158359ce9399326386e7)

   Yeah, most Schemes only handle double floats except in vectors, come to think of it.

Comments are closed.
