---
title: Modern Dataflow Frameworks Wanted
url: https://blog.regehr.org/archives/811
published: "2012-09-30T22:09:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=811
---

# Modern Dataflow Frameworks Wanted

Dataflow analysis, or static analysis, is a way to compute properties that hold over all possible executions of a program. For example, if a compiler can conclude that an expression always evaluates to the value 5, this fact can be used to avoid computing the expression at runtime.

The math behind dataflow analysis is probably one of the ten most important results in theoretical computer science. It supports a strong separation of concerns where a generic dataflow framework takes care of the bookkeeping and a collection of transfer functions implement the specific analysis.

Once a framework exists, writing analyses is easy and entertaining. It’s great to form a little hypothesis like “we could get rid of many array bounds checks by keeping track of integer intervals,” write a few transfer functions, and then analyze some large programs in order to validate or disprove the hypothesis. On the other hand, writing a dataflow analysis without a framework is a hellish amount of work.

I recently read [this paper](http://clip.dia.fi.upm.es/~jorge/docs/wrapped-intervals-aplas12.pdf) which evaluates a new idea for analyzing integer ranges. The authors implemented their analysis over the LLVM IR, which is nice since LLVM serves as a partial framework, permitting them to analyze some of the SPEC benchmarks. On the other hand, significant limitations of their analysis (intraprocedural, limited analysis of globals and pointed-to values) would seem to indicate that LLVM does not yet serve as a great framework for this sort of research. The result is that even though the authors of this paper have done the right thing, their evaluation remains tantalizing at best. My former student Nathan Cooprider had similar problems implementing dataflow analyses using [CIL](http://sourceforge.net/projects/cil/), which saved us probably 18 months of work and made his project feasible. Even so, CIL had numerous shortcomings as a dataflow framework and a lot of effort was required to get things working.

A good dataflow framework should:

1. Contain front-ends for one or more real programming languages.
2. Be open source and well documented.
3. Expose a clean intermediate representation.
4. Make easy things easy. A user should have to write only a minimal amount of code to support a simple static analysis such as detecting null pointers or determining signedness of integers.
5. Make it easy to write at least one kind of static analysis. For example, an integer range analysis is quite different from a live variables analysis, and both are very different from a pointer analysis.
6. Provide easy access to source-level information such as lines, columns, variable names, etc.
7. Heavily optimize the program prior to analysis. One of the easiest ways to increase the precision of an analysis is to run it after a comprehensive collection of optimization passes (such as those supported by LLVM) has removed dead code, useless indirection, and other junk that tends to pollute analysis results.
8. Permit analyses to learn from the results of other analyses.
9. Provide good support for turning analysis results into assertions; Nathan probably couldn’t have worked the bugs out of his analyzers without doing this.
10. Provide efficient fixpoint iteration algorithms.
11. Make it easy to write transformations that consume analysis results.
12. Provide transparent support for interprocedural and whole program analysis.

It is possible that [Soot](http://www.sable.mcgill.ca/soot/) and [Rose](http://rosecompiler.org/) meet all of these requirements. CIL does not. I don’t think LLVM is too far off. It seems likely that good analysis frameworks for functional languages exist, but I’ve paid no attention.

One of the biggest holes that I see right now is a good unified framework for static analysis of languages such as JavaScript, Python, Perl, and PHP. The heavy reliance on strings, hashes, and dynamic types in these languages makes them a lot different (and in some ways a lot more difficult) than traditional compiled languages. I’m not totally sure that the differences between these languages can be abstracted away in a clean fashion, but it seems worth trying.

**UPDATE:**

Pascal Cuoq points out the [Frama-C](http://frama-c.com/) meets all of the criteria at least to some extent. Ed Schwartz points to [CPAChecker](http://cpachecker.sosy-lab.org/) which looks interesting though I know nothing about it.

An important criterion that I forgot to list is the ability of a dataflow framework to provide knobs for tuning precision against resource usage. Examples of this include configurable widening thresholds and tunable degrees of path and context sensitivity.
