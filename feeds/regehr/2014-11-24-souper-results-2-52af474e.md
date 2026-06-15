---
title: Souper Results 2
url: https://blog.regehr.org/archives/1192
published: "2014-11-24T20:35:34Z"
feed: regehr
guid: http://blog.regehr.org/?p=1192
---

# Souper Results 2

The [Souper](https://github.com/google/souper) superoptimizer has made some progress since my [last post about it](https://blog.regehr.org/archives/1146).

We wrote compiler drivers that usually reduce the problem of building a project with Souper to `make CC=sclang CXX=sclang++`. Souper now uses Redis to cache optimizations so that even if the initial build of a program using Souper is slow, subsequent builds will be pretty fast. We fixed several problems that were preventing Souper from building largish programs like LLVM and GCC. This works now and, as far as we know, Souper can be used to optimize arbitrary LLVM code.

Souper now understands the ctpop, ctlz, cttz, and bswap intrinsics. It no longer generates only i1 values, but rather synthesizes constant values of any width. Constant synthesis is not fast and it requires a solver with good support for quantifiers, currently only Z3 (synthesizing constants without quantifiers isn’t hard, we just haven’t implemented that yet). Here’s a list of [constants synthesized while building LLVM](http://www.cs.utah.edu/~regehr/constants.txt) with Souper. The left side of each line is the number of times the constant on the right side was synthesized. i1 constants dominate but it’s fun to see, for example, that Souper was able to synthesize the 64-bit value 90112 four times. Where did that come from?

Souper has two main use cases. First, application developers can use Souper directly to optimize code they are compiling. Second, LLVM developers can use Souper to learn about optimizations missed by the existing optimization passes. We’re trying to make it useful to both of these audiences.

To make Souper more useful for compiler developers, we implemented a C-Reduce-like reducer for Souper optimizations. This is necessary because Souper extracts and attempts to optimize pieces of LLVM that are as large as possible, meaning that its optimizations often contain extraneous material. A reduced optimization has the handy invariant that no path condition, UB qualifier (nsw, nuw, exact), or leaf instruction can be removed without breaking the optimization. We did some cross-checking between Souper and [Alive](https://github.com/nunoplopes/alive), as a sanity check on both tools. Additionally, we convert each Souper optimization back into LLVM and run it through opt -O3 in order to weed out any optimizations that LLVM already knows how to do. For example, Souper loves to prove that `icmp eq %0, %0` can be simplified to 1. This is not useful.

While building LLVM, ~16,000 Souper optimizations fire. Some of these optimizations are duplicates (presumably due to inlining and header inclusion); ~7000 of them are distinct. After reduction there are ~4000 distinct optimizations and LLVM does not know how to perform ~1500 of them. Even 1500 optimizations is lot of work to look through and of course not all of them matter. To help figure out which optimizations matter, we implemented two kinds of optimization profiling. The first is *static profiling*, which counts the number of times an optimization is applied at compile time. Implementing optimizations with a high static profile count would tend to reduce the size of the compiler’s generated code. Second, we implemented *dynamic profiling*, which counts the number of times each optimized piece of code is executed. This is accomplished by instrumenting the compiled program so that it dumps dynamic profile information to a Redis server using an atexit() handler. Implementing optimizations with a high dynamic profile count would tend to decrease the runtime of generated code. Of course, all standard caveats about profile-driven optimization apply here. Also keep in mind that Souper is extremely specific while compilers are less so: there is a many-to-one relationship between optimizations discovered by Souper and optimizations you would implement in LLVM. Therefore, it may well be the case that there are collections of low-ranking Souper optimizations that would rank highly if considered as a group, and that could all be implemented by a single LLVM transformation. We’ve experimented a bit with trying to automatically aggregate similar Souper optimizations, but so far I haven’t been too happy with the results.

If we take a Souper-optimized LLVM and use it to build SPEC CPU 2006, this is the optimization with the highest dynamic profile count; it is executed ~286 million times:

**```** **%0:i64 = var** **%1:i64 = and 15:i64, %0** **%2:i1 = eq 0:i64, %1** **pc %2 1:i1** **%3:i64 = and 7:i64, %0** **%4:i1 = eq 0:i64, %3** **cand %4 1:i1**

**```**

The first four lines tell us that the arbitrary 64-bit value %0 is known to have zeros in its four bottom bits. The last three lines tell us that — of course — %0 has zeros in its three bottom bits. LLVM doesn’t understand this yet, leading to a lot of unnecessary conditional jumps.

Here’s the collection of Souper optimizations that are discovered while building LLVM/Clang/Compiler-RT r222538:

- [sorted by decreasing dynamic profile count](https://blog.regehr.org/extra_files/souper-nov-14/bydprofile.html)
- [sorted by decreasing static profile count](https://blog.regehr.org/extra_files/souper-nov-14/bysprofile.html)
- [sorted by increasing size](https://blog.regehr.org/extra_files/souper-nov-14/bysize.html)

The clang binary from a “Release” build with Souper is about 800 KB smaller than the clang built without Souper. Please let us know about any bugs in the output above, including missed optimizations (but don’t tell us about missing vector, FP, or memory optimizations, we know that those are not supported yet). In the course of this work Raimondas ran across [a Z3 bug](http://z3.codeplex.com/workitem/143); luckily he caught it by cross-checking Souper’s results using a different solver, instead of having to debug the resulting miscompilation.

The main thing that Souper does not do, that you would expect a superoptimizer to do, is to synthesize sequences of instructions. Much of our work over the last six months has been building infrastructure to support instruction synthesis, and almost all of that is now in place. Synthesis is our next major piece of work.

In the meantime, [Peter has run Souper over libgo](https://github.com/go-llvm/llgo/pull/185). I would like to build something a bit bigger such as Chromium. If you have a recipe for that, please drop me a line. I got as far as noticing that Chromium builds its own LLVM at which point my allergy to build systems kicked in. Integrating Souper into a build of the Rust compiler might also produce interesting results; it should be as easy as starting Redis and making sure our opt pass gets loaded in the right places.

Souper is by Peter Collingbourne at Google, by my postdoc Raimondas Sasnauskas, by Yang Chen at nVidia, by my student Jubi Taneja, by Jeroen Ketema at Imperial College London, and by me.
