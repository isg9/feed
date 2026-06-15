---
title: The Future of Compiler Optimization
url: https://blog.regehr.org/archives/247
published: "2010-08-18T20:16:28Z"
feed: regehr
guid: http://blog.regehr.org/?p=247
---

# The Future of Compiler Optimization

Also see [The Future of Compiler Correctness](https://blog.regehr.org/archives/249).

Compiler optimizations are great: developers can write intuitive code in high-level languages, and still have them execute in a reasonably fast way. On the other hand, progress in optimization research is excruciatingly slow despite hundreds of papers being published on the topic every year. Proebsting’s Law speculates that improvements in optimization will double the performance of a program in 18 years — a humorous reference to the better-known Moore’s Law where the time constant is only 18 months. Of course, good optimization research often focuses not so much on speeding up dusty-deck codes, but rather on taking programming idioms previously thought to be impossibly slow — infinitely ranged integers, for example — and making them practical.

This piece is about the future of compiler optimization: some areas that I predict will be interesting and relevant during the next 25 years or so. At the bottom are a few anti-predictions: areas I think are red herrings or played out.

# Verification and Optimization Will Join Forces

Currently, optimization tools are totally divorced from verification tools. This is odd because both kinds of tools are doing more or less the same thing: attempting to compute and make use of program properties that are true over all possible executions. Let’s contrast the two approaches:

**Optimizer:** Attempts to infer program properties from source code in a sound way, so that semantics-preserving program transformations can be performed. Since optimizers must run quickly and get no help from developers, it is very uncommon for deep properties to be discovered. Rather, optimizers uncover low-level and superficial facts about programs: “w is always constant” or “y and z do not alias.”

**Verifier:** Starts with high-level program properties specified by developers and attempts to show that they are consistent with the code. Runs on the unoptimized program text, so must repeat much of the work done by the optimizer. The high-level properties and intermediate mid-level properties are unavailable for purposes of generating better code.

So in one case the properties are high-level and given, and they are checked. In the other case they are low-level and inferred, and are used to drive transformations. I predict that moving forward, it will turn out to be extremely useful to have interplay between these. Here are a few examples:

- Verification often relies on separation arguments: claims that two pieces of code access non-overlapping parts of the machine’s state. Separation supports a variety of optimizations including deterministic parallel execution.
- Showing that a computation executes without causing any exceptional conditions (divide by zero, null pointer access, etc.) is an important part of verification. These properties can also be used to optimize, for example by making it easier to show that computations can be safely reordered, eliminated, or duplicated.
- Designing algorithms and data structures for hashing, searching, sorting, and the like is not very entertaining or productive. One good way to avoid this is the Perl approach: create sophisticated and well-tuned algorithms and just use them everywhere. In the non-scripting domain, a bit more finesse is required. If I’ve properly annotated my code to support verification, the properties (stability, etc.) that I expect from my sorting algorithms will be available to the compiler. This, probably combined with profile data, can be used to choose the best sort implementation for my program. Applied at multiple levels of abstraction — but mostly behind the scenes as part of the runtime library — algorithm selection will be a useful technique for increasing developer productivity and application performance. It will be particularly important in parallel and distributed programming. There has been work on automated algorithm selection, but nothing I’ve read has been convincing. Probably this is a bit of a thankless research area: it won’t give massive speedups on any particular application, but rather will make like slightly to moderately easier for a large number of programmers.

As a side benefit, specification-driven optimizers will help convince developers to write the specs in the first place. Currently, it’s hard to convince a substantial fraction of developers that serious testing is a good idea, much less formal specification of properties of their code. However, people love to write fast code and if it can be shown that specifications help the optimizer, specification will gain traction.

Am I saying that in the future there will be a single tool that both optimizes and verifies code? While that is an interesting idea, it’s not likely in the short term. A more practical way to permit these tools to interoperate is to define a data format so they can exchange information about program properties.

# Decision Procedures Will Be Heavily Used

Decision procedures for boolean satisfiability and SMT problems have become very powerful during the last 10 years. The basic idea is that many interesting problem instances — even when the general case is intractable or undecidable — can be automatically decided in practice. Verifying a program requires answering lots of (often silly, but occasionally quite deep) questions and decision procedures have emerged as a key way to get these answers in a relatively easy way. Of course, optimizers also ask a lot of extremely silly questions about the programs they’re optimizing and decision procedures can help there as well. The reason decision procedures are not currently used much in compilers is that fast compile time is a top-priority goal.

A great example of using a decision procedure in an optimizer is the [peephole superoptimizer](http://theory.stanford.edu/~aiken/publications/papers/asplos06.pdf), which uses a SAT solver to answer questions of the form “Given this sequence of x86 instructions, is there a shorter sequence that has the same effect?” Computing the answer may be slow, but the results can be stored in a lookup table, supporting rapid reuse. The results in the peephole superoptimizer paper were not very impressive: basically they did a fine job automatically generating known optimizations, but didn’t discover a lot of new optimizations.

I believe a much more effective superoptimizer could be created by:

- Operating on small (~5 nodes) sub-graphs of the program dependency graph, as opposed to short linear sequences of instructions. Sequences of instructions often mix irrelevant computations, whereas sub-graphs of the PDG are dependent by definition, and therefore are likely to encode interesting, optimizable computations.
- Operating at the level of an intermediate representation such as LLVM, as opposed to assembly code. First, more high-level information is preserved (is overflow behavior defined? is the memory access volatile?). Second, register allocation, instruction scheduling, and other target-specific optimizations have not yet been performed. The superoptimizer is likely to mess these up, or at least to be constrained by them in incidental ways. Third, the non-superoptimized optimization passes can clean up any left-over junk, getting a bit of extra mileage.
- Harnessing specifications, when available. The peephole superoptimizer effectively uses x86 instructions as the specification, which forces incidental implementation decisions to be fixed in stone, limiting the effectiveness of the tool. If I have a formal specification of the hash function I want to implement, we might as well start from there instead of committing to an implementation and then trying to optimize it. The [Denali superoptimizer](http://www.hpl.hp.com/techreports/Compaq-DEC/SRC-RR-171.pdf) was based on this idea, but their article fails to report any nontrivial results. The approach seems reasonable and the paper doesn’t admit to any fatal flaws, so we’re left to guess if they simply decided to stop working on Denali too early, or if there’s a major hidden problem. It could easily be that their approach works much better now simply because the decision procedures are a decade better.

My random guess is that about half of LLVM’s optimizations could be superseded by a strong superoptimizer. This would have major advantages in code maintainability, and the superoptimizer would keep getting stronger as time went by, since it would always be learning new tricks.

Decision procedures can be used in other parts of compilers; this is perhaps just the most obvious one.

# Compilers Will Rely on Models and Feedback

Existing compilers are based on simple hard-coded heuristics for applying optimizations: “perform inline substitution of a function when its body is less than 17 instructions, unless it is called from more than 25 sites.” Sometimes, these work perfectly well; for example, it never hurts to replace an always-constant variable with the constant. On the other hand, many little decisions made by a compiler, such as which loops to unroll, are not well-suited to simple heuristics. In these cases, the compiler needs an answer to the question: is this transformation a win? The solution, unfortunately, is a bit complicated. First, we either need a model of how the optimization works, or else we can just try it and see — but being ready to roll back the transformation if it’s not profitable. Second, we need either a model of the target platform or else we can just run our code directly and see if it works better. Third, we require a model of the system developers’ preferences: did they want small code? Fast code? This model will be trivial in many cases, but it can be interesting in the embedded world where I may absolutely require my code size to be less than 32 KB (since that is how much ROM I have) but not really care if it’s 1 byte less or 10 KB less. Does there exist an embedded compiler today that can operate on this kind of high-level constraint? Not that I know of.

My prediction is that a compiler that makes pervasive use of models and feedback will be able to generate significantly better code than today’s compilers, even using only the optimization passes that are currently available. The benefit comes from making thousands of correct decisions about which optimizations to apply, instead of making 80% good decisions and being slightly to totally wrong the rest of the time. For platforms with interesting performance characteristics and for embedded systems with hard resource constraints, the benefits may be significant.

# Optimizations Will Emit Proof Witnesses

To support routine compiler debugging as well as more ambitious goals such as translation validation, it will become more common to implement optimization passes that emit proof witnesses: data that can be used to build proofs that the passes didn’t change the meaning of the code while transforming it. Currently this is painful, but a variety of technologies will make it easier. Any optimizer based on an SMT solver can take advantage of proof-producing SMT. Research by people like [Sorin Lerner](http://cseweb.ucsd.edu/~lerner/) is producing infrastructure for creating proof-producing optimizations.

# A Few IRs Will Win

A compiler is built around its intermediate representations (IRs). Ten years ago, it didn’t seem clear that the “IR to rule them all” was achievable. This still isn’t totally clear, but for many purposes LLVM looks good enough. It adequately fills the low-level niche. A few more IRs, perhaps one for GPU-style computing and one that can capture interesting properties of functional languages, are needed, but not very many. Substantial engineering wins will be obtained if the compiler community centralizes its efforts around a handful of well-designed IRs. This will largely happen and nobody will ever write an x86 register allocator again, unless they really feel like it.

# Anti-Predictions

There will be few, if any, major improvements in the speedup provided by the optimizations that have historically been most important: register allocation, alias analysis, function inlining, etc. The engineering of register allocators and such will no doubt improve, but the quality of the generated code will not.

The distinction between online (JVM style) vs. offline (C++ compiler style) optimization is not fundamental and will not seem like a big deal in the long run. Rather, large collections of optimizers will be built around IRs like LLVM, and people will be able to create AOT and JIT compilers, as well as link-time optimizers and whatever else seems useful, simply by choosing suitable components from the toolkit.

Finally — and this is a prediction that might actually be wrong, though I don’t think so — machine learning in compilers will not end up being fundamental. Machine learning permits a model to be inferred from a collection of data. To be sure, there are parts of the compiler (phase ordering, tuning the inliner, etc.) where we cannot be bothered to build a good model by hand or from first principles, and so machine learning will be used. However, I predict that the benefits coming from these techniques will always be low-grade. To take an example, consider phase ordering. People have shown great speedups in for example DSP codes using machine learning. However, what is really being bought is fast compile times: if I’m willing to wait a bit, I could always get a good optimization result by running all of my optimization passes until a fixpoint is reached. Similarly, I might be able to get great optimization results by using machine learning to tune my function inlining pass. However, if I’m willing to wait a bit, I can always get the same or better results by speculatively inlining, and then backing out from decisions that don’t seem to be working well.
