---
title: Translation Validation of Bounded Exhaustive Test Cases
url: https://blog.regehr.org/archives/1510
published: "2017-05-09T16:04:34Z"
feed: regehr
guid: http://blog.regehr.org/?p=1510
---

# Translation Validation of Bounded Exhaustive Test Cases

*This piece is jointly authored by [Nuno Lopes](http://web.ist.utl.pt/nuno.lopes/) and John Regehr.*

Compilers should be correct, but it is not straightforward to formally verify a production-quality compiler implementation. It is just too difficult to recover the high-level algorithms by looking at an enormous mess of arithmetic, loops, and memory side effects. One solution is to write a new compiler such as CompCert that is designed to be verified. Alternatively, we keep our large, low-level code base such as GCC or LLVM and settle for weaker forms of validation than formal verification. This piece is about a new way to do the second thing. Our focus is the middle-end optimizers, which seem to be the most difficult part of a compiler to get right. The target is LLVM.

End-to-end compiler testing, supported by a random source code generator like Csmith, is great — but it only gets us so far. The expressiveness of the program generator is one limitation, but a more serious problem is the normalization that happens in the compiler frontend. The issue is that there are a lot of valid code patterns that Clang will never emit and that are therefore impossible to test by driving Clang. As a Clang user you may not happen to care about this, but as LLVM people we want the middle-end optimizations to be free of logic errors and also the non-Clang-emittable code is important in practice since there are lots of frontends out there besides Clang.

The first step is to generate lots of LLVM IR. Rather than creating a relatively small number of large functions, as Csmith would do, [this IR generator](https://github.com/regehr/opt-fuzz) generates lots of tiny functions: it uses bounded exhaustive test generation to create every LLVM function up to a certain size. A fun thing about this kind of generator is its choose() operator. In random mode, choose() returns a random number; in exhaustive mode, it uses fork() to explore all alternatives. While this isn’t the most efficient way to do search, leveraging the OS keeps the generator very simple. The most vexing thing about this design is allowing it to use multiple cores while stopping it from being a fork bomb. The current version doesn’t contain code that tries to do this.

The next step is to run LLVM optimizations on the generated functions. One thing we want to try is the collection of passes that implements “-O2,” but it also makes sense to run some individual passes since it is possible for sequences of passes to miss bugs: early passes can destroy constructs that would trigger bugs in later ones, and the late passes can clean up problems introduced by earlier ones. In fact both of those things seem to happen quite often.

We end up with lots of pairs of unoptimized and optimized LLVM functions. The obvious thing to do is run them with the same inputs and make sure that the outputs are the same, but that only works when the executions encounter no undefined behaviors. Solutions to the UB problem include:

1. Generating UB-free code, as Csmith would. At the level of these tiny functions that would be a major handicap on the generator’s expressiveness and we definitely do not wish to do it.

2. Create an LLVM interpreter that detects UB instead of silently skipping over it. The rule is that the optimizer is allowed to preserve UB or remove it, but never to add it. In other words, the correctness criterion for any compiler transformation isn’t input/output equivalence but rather input/output refinement. Someone needs to write this interpreter, perhaps using [lli](http://llvm.org/docs/CommandGuide/lli.html) as a starting point (though the last time we looked, the slow/simple interpreter mode of lli had suffered some bit rot).

3. Formally verify the refinement relation using [Alive](http://www.cs.utah.edu/~regehr/papers/pldi15.pdf). This is better than an interpreter because Alive verifies the optimization for all inputs to the function, but worse because Alive doesn’t support all of LLVM, but rather a loop-free subset.

It is option three that we chose. The Alive language isn’t LLVM but rather an LLVM-like DSL, but it is not too hard to automatically translate the supported subset of LLVM into Alive.

In the configuration that we tested (2- and 4-bit integers, three instructions per function, including select but not including real control flow, floating point, memory, or vectors) about 44.8 million functions are generated and binned into 1000 files. We identified seven configurations of the LLVM optimizers that we wanted to test: -O2, [SCCP](http://llvm.org/docs/Passes.html#sccp-sparse-conditional-constant-propagation), [GVN](http://llvm.org/docs/Passes.html#gvn-global-value-numbering), [NewGVN](http://lists.llvm.org/pipermail/llvm-dev/2016-December/108503.html), [Reassociate](http://llvm.org/docs/Passes.html#reassociate-reassociate-expressions), InstSimplify, and [InstCombine](http://llvm.org/docs/Passes.html#passes-instcombine). Then, to make the testing happen we allocated 4000 CPU cores (in an Azure HPC cluster) to process in batch the 7000 combinations of file + optimization options. Each combination takes between one and two hours, depending on how many functions are transformed and how long Alive takes to verify the changes.

If we could generate all LLVM functions and verify optimization of them, then we’d have done formal verification under another name. Of course, practically speaking, there’s a massive combinatorial explosion and we can only scratch the surface. Nevertheless, we found bugs. They fall into two categories: those that we reported and that were fixed, and those that cannot be fixed at this time.

We found six fixable LLVM bugs. The most common problem was transformations wrongly preserving the nsw/nuw/exact attributes that enable undefined behaviors in some LLVM instructions. This occurred with InstCombine \[ [1](https://bugs.llvm.org/show_bug.cgi?id=31633)\], GVN \[ [1](https://bugs.llvm.org/show_bug.cgi?id=23922)\], and Reassociate \[ [1](https://bugs.llvm.org/show_bug.cgi?id=23675), [2](https://bugs.llvm.org/show_bug.cgi?id=23926)\]. InstSimplify generated code that produces the wrong output for some inputs \[ [1](https://bugs.llvm.org/show_bug.cgi?id=32949)\]. Finally, we triggered a crash in llc \[ [1](https://bugs.llvm.org/show_bug.cgi?id=23664)\].

The unfixable bugs stem from problems with LLVM’s undefined behavior model. One way to fix these bugs is to delete the offending optimizations, but some of them are considered important. You might be tempted to instead fix them by tweaking the LLVM semantics in such a way that all of the optimizations currently performed by LLVM are valid. We believe this to be impossible: that there does not exist a useful and consistent semantics that can justify all of the observed optimizations.

A common kind of unfixable bug is seen in the simplification logic that LLVM has for [select](http://llvm.org/docs/LangRef.html#select-instruction): it transforms “select %X, undef, %Y” into “%Y”. This is incorrect (more details in the post linked above) and, worse, has been shown to trigger end-to-end miscompilations \[ [1](https://bugs.llvm.org/show_bug.cgi?id=31633)\]. Another source of problems is the different semantics that different parts of LLVM assume for branches: these can also cause end-to-end miscompilations \[ [1](https://bugs.llvm.org/show_bug.cgi?id=31632), [2](https://bugs.llvm.org/show_bug.cgi?id=31652)\].

In summary, this is a kind of compiler testing that should be done; it’s relatively easy and the resulting failing test cases are always small and understandable. If someone builds an UB-aware LLVM interpreter then no tricky formal-methods-based tools are required. This method could be easily extended to cover other compilers.

There are some follow-on projects that would most likely provide a good return on investment. Our test cases will reveal many, many instances where an LLVM pass erases an UB flag that it could have preserved; these could be turned into patches. We can do differential testing of passes against their replacements (for example, NewGVN vs. GVN) to look for precision regressions. The set of instructions that we generate should be extended; for example, opt-fuzz already has some limited support for control flow.

The code to run these tests is [here](https://github.com/nunoplopes/alive/tree/newsema/tv).
