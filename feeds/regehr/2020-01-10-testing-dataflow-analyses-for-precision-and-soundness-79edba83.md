---
title: Testing Dataflow Analyses for Precision and Soundness
url: https://blog.regehr.org/archives/1709
published: "2020-01-10T21:18:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=1709
---

# Testing Dataflow Analyses for Precision and Soundness

\[This piece is co-authored by Jubi Taneja, Zhengyang Liu, and John Regehr; it’s a summary of some of the findings from [a paper that we just recently completed the camera ready copy for](http://www.cs.utah.edu/~regehr/cgo20.pdf), that is going to be published at [CGO (Code Generation and Optimization) 2020](https://cgo-conference.github.io/cgo2020/).\]

**Update from Jan 12 2020:** Looks like there’s [a patch in the works](https://reviews.llvm.org/D72573) fixing some imprecisions we found!

Optimizing compilers use dataflow analyses to compute properties that are true in every execution of the program being compiled. For example, if an array access is provably in-bounds in every execution, the bounds check can be elided. A dataflow analysis should be:

- **Sound**: It must never erroneously claim that a property holds. Unsound dataflow analyses typically lead to miscompilations, for example by helping eliminate a bounds check that could fail.

- **Precise**: When the property holds, the dataflow analysis should be able to see this. The halting problem tells us that this is not always possible, but we’d like the analysis to mostly work pretty well in practice.

- **Fast**: We don’t want dataflow analysis to slow down the overall compilation very much.

Meeting these constraints isn’t easy, and moreover compiler developers generally attempt to do this in a seat-of-the-pants fashion, without any kind of help from formal-methods-based tools. Our paper is based on the premise that formal methods can help! Using an SMT solver, we can create dataflow analyses that are sound and maximally precise: they never erroneously claim that the property holds and they always conclude that the desired property holds when this can be done at all by the analysis in question. (But keep in mind these claims require an ideal SMT solver: solver bugs and solver timeouts respectively defeat them.)

Our work is only about soundness and precision: it does not help make dataflow analyses faster. Moreover, since it involves lots of solver calls, it is slow: we can use it offline to find places where the compiler might want to be improved, but it is not possible to just drop it into the compiler as a replacement for existing dataflow analyses. (We do have some preliminary work on synthesizing dataflow implementations, and will write about that some other time.)

The first dataflow analysis we’ll look at is one of LLVM’s workhorses: known bits. This analysis tries to prove that individual bits of SSA values are either 0 or 1 in all executions. Many cases handled by this analysis are easy to understand. For example, a value that has been zero-extended obviously contains zeroes in its high-order bits. Similarly, a value that has been orred with a bitmask will have some ones in it. Other cases are harder. As an exercise, you might try to work out the known bits in the result of a subtraction or a signed division.

Known bits has a variety of customers in LLVM middle-ends and backends. For example, the “simplify CFG” pass computes the known bits in the argument to a [switch instruction](https://llvm.org/docs/LangRef.html#switch-instruction) in order to optimize away dead switch cases; a switch case is provably dead if at least one bit of its value conflicts with a known bit in the value being switched on.

Ok, it’s time for an example. Consider this LLVM IR which shifts the value four to the right by the number of bit positions specified in its argument value:

**`** **define i32 @foo(i32) {** **%a = lshr i32 4, %0** **ret i32 %a** **}** **`**

As of January 2020, LLVM doesn’t return any known bits for %a. If you would like to see this for yourself, first build a fresh LLVM and then second build [this out-of-tree pass](https://github.com/regehr/llvm-dataflow-info) which calls LLVM’s dataflow analysis and pretty-prints its results. Run it like so:

**`** **$ opt -load ./LLVMDataflowInfo.so -dataflow-info ../test/test1.ll -o /dev/null** **function: foo** **%a = lshr i32 4, %0** **known: ????????????????????????????????** **isKnownToBePowerOfTwo = 0** **LVI: [0,5)** **SCEV: %a full-set full-set** **demanded: 11111111111111111111111111111111** **$** **`**

It is easy to see that a better answer is possible: LLVM could have returned `00000000000000000000000000000???`. In other words, the result will always have 29 zeroes in the high end. This is the best possible result: a sound implementation of known bits cannot prove anything about the last three bits. This is because, for each of these bits, there exists an execution in which the bit is zero and also an execution in which the bit is one.

Now let’s switch gears and look at a different dataflow analysis from LLVM: integer ranges. This analysis lives in the “Lazy Value Info” analysis pass and it computes one of four results for an SSA value; the one we care about right now is a “regular range” \[a, b) where a is smaller (using unsigned comparison) than b. This result means that in every execution of the program, the SSA value will be at least a and also less than b, again using unsigned comparisons.

For example, consider this IR which zero-extends its 8-bit arguments to 64 bits, xors the extended values, and returns the result:

**`** **define i64 @foo(i8, i8) {** **%x = zext i8 %0 to i64** **%y = zext i8 %1 to i64** **%z = xor i64 %x, %y** **ret i64 %z** **}** **`**

It is obvious that \[0, 256) is the result we want for %z: no smaller range captures all of the possibilities, and there are no possible executions that require a larger range. LLVM, on the other hand, returns a special “full set” result for %z, meaning that it is unable to compute any useful information about it. This happens because [the code for modeling the effect of binary operators on integer ranges does not handle xor at all](https://github.com/llvm/llvm-project/blob/085898d469ab782f0a26f119b109aa8eb5d37745/llvm/lib/IR/ConstantRange.cpp#L780-L816).

In our paper we deal with some additional dataflow analyses, such as “demanded bits,” which is LLVM’s funny name for a don’t-care analysis that looks for bits whose value does not affect subsequent computations. Demanded bits is interesting because it propagates information backwards along SSA edges. We won’t cover demanded bits or any of the others in this post.

Ok, now let’s get to the good stuff: computing maximally precise dataflow results. We build on Souper, since it already knows how to ask a solver questions about LLVM code, but besides that we don’t make interesting use of Souper’s capabilities. The bad news is that we don’t know how to just plug in a formal definition of a dataflow analysis and get answers directly from that. Rather, we need to spend a bit of time studying the structure of each abstract domain that we target, in order to create a solver-based algorithm for computing maximally precise results reasonably efficiently.

For known bits, we can deal with each bit individually. For a given bit, we’ll first ask the solver whether there exists any valuation of the inputs that can force the bit to be zero. If this returns UNSAT, then we have just proven that this bit is one in every execution. If this query is SAT, we issue a second, analogous, query asking whether this bit can ever be one. If not, it is known to be zero. If neither query returns UNSAT, we know that this bit can be both 0 and 1, and we move on to the next bit. This procedure gives the most precise result using at most 2n solver queries (for a value n bits wide). The gist of the argument for maximal precision is that known bits are independent of each other. This is a nice result because a naive search of all elements of the known bits lattice would require 3n solver calls and this is clearly infeasible.

Looking back at the first example above, we can issue these commands:

**```** **$ llvm-as test1.ll** **$ souper $SOUPER_SOLVER --infer-known-bits test1.bc** **; Listing valid replacements.** **; Using solver: Z3 + internal cache**

**; Function: foo** **%0:i32 = var** **%1:i32 = lshr 4:i32, %0** **infer %1** **; knownBits from souper: 00000000000000000000000000000xxx** **$**

**```**

Integer ranges, it turns out, are more difficult. First, due to a “wrapped range” feature that we’ll not discuss here (again, see our paper) this abstract domain doesn’t form a lattice, so the definition of “maximal precision” becomes problematic. We pave over this issue by relaxing the notion of maximal precision a bit. Instead of insisting on the unique maximally precise result that would have to exist if the domain had been a lattice (but does not, since it isn’t), we’ll accept any result as maximally precise if there does not exist an integer range that excludes a larger number of values than is excluded by the given range. (If you find static analysis using non-lattice abstract domains interesting, [this very nice paper](https://jorgenavas.github.io/papers/nonlattice-sas13.pdf) contains more details.) The second problem with integer ranges is that there’s no obvious bit-level divide and conquer strategy for finding the maximally precise result. We’ll need a different approach.

Ignoring a special “empty set” value which indicates dead code or unconditional undefined behavior, we’re looking for an integer range \[x, x+c) such that c is minimized. For example, we can start with c=1, corresponding to the hypothesis that there is a valid dataflow result \[x, x+1), which excludes all integer values but one. In other words, this hypothesis corresponds to the SSA value being a constant, x. But how can we find the concrete value of x in a large search space such as 264? We do this using solver-based constant synthesis. If synthesis succeeds, we have found the maximally precise result. If no x exists such that \[x, x+1) is a valid dataflow result, we launch a binary search to find the smallest c for which a suitable x does exist. This range will satisfy the maximal precision condition from the previous paragraph. The details of synthesizing an integer value meeting constraints don’t matter here as long as we have [a procedure that works](https://blog.regehr.org/archives/1636).

Going back to the second example:

**```** **$ llvm-as test2.ll** **$ souper $SOUPER_SOLVER --infer-range test2.bc**

**... a few irrelevant lines elided ...**

**; Function: foo** **%0:i8 = var** **%1:i64 = zext %0** **%2:i8 = var** **%3:i64 = zext %2** **%4:i64 = xor %1, %3** **infer %4** **; range from souper: [0,256)** **$**

**```**

So in both of these cases we get the obvious maximally precise result. However, these examples are contrived. What code should we look at, when doing this work for real? We looked at all of the bitcode from compiling the C and C++ programs in SPEC CPU 2017, but really any large collection of LLVM IR should be fine. For each SSA value, we use our code and LLVM’s to compute dataflow results and then compare them. The possibilities are:

- Our result is more precise than LLVM’s: lost precision, flag for human inspection.

- LLVM’s result is more precise than ours: LLVM is unsound, flag for human inspection.

- Results have the same precision: ignore.

Across the LLVM IR from compiling SPEC CPU 2017, we find many examples where LLVM’s dataflow analyses leave some precision on the floor. Is it worth making the analyses more precise? This is subjective, and can only be answered by compiler developers. Overall — despite the fact that we find a lot of imprecision — it looks to us like LLVM’s dataflow analyses are quite mature and provide a lot of precision where it matters. This would not have been the case around 2010, but these analyses have gotten a lot of tweaks since then. Even so, there is still plenty of room for improvement.

We thought that while doing this work we might find examples where LLVM’s dataflow result is “more precise” than our maximally precise one. This would indicate soundness errors in LLVM. We did not find any such examples, but we did re-introduce some old soundness bugs in LLVM into the version we used for testing, in order to verify that we can automatically detect these. Details are in the paper.

Since our method requires concrete code fragments as the basis for comparing the precision of LLVM’s analyses and ours, we might ask: is this is an advantage or a disadvantage? On the plus side, we’ll find imprecisions and soundness errors that occur when compiling real programs, and presumably these are the ones that should be a higher priority for fixing. On the other side, we’ll miss cases not exercised by the test programs.

In summary, we have presented several solver-based algorithms for computing maximally precise dataflow results, with the goal of spotting soundness errors and lost precision in a real compiler, LLVM. This is part of a [larger research program outlined here](https://arxiv.org/pdf/1809.02161.pdf). The idea is to move compiler construction away from the current situation, where compilers are huge collections of bespoke code, and towards a situation where the mathematical foundations of compilation can be directly used to support their construction.
