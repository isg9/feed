---
title: Randomly Testing a Static Analyzer
url: https://blog.regehr.org/archives/680
published: "2012-02-03T16:43:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=680
---

# Randomly Testing a Static Analyzer

Static analyzers are intended to find bugs in code, and to show that certain kinds of bugs don’t exist. However, static analyzers are themselves large, complicated programs, leading to a “who watches the watchmen” problem. Pascal Cuoq, one of the people behind the excellent [Frama-C](http://frama-c.com/) analyzer, took it upon himself to run the fuzz-fix cycle for Frama-C and [Csmith](http://embed.cs.utah.edu/csmith/) all the way to its fixpoint — where no bugs can be found.

Fuzz testing relies on knowing some invariants for the system under test; often, this is just the trivial “program shouldn’t crash.” Luckily, for compiler testing we can do much better and Pascal did some hacking to turn Frama-C into a (relatively) efficient interpreter for C programs, making it possible to compare Frama-C’s interpretation of a program against regular C compilers.

The aspect of this exercise that I found most interesting was that Frama-C found some nasty bugs in Csmith at the same time that Csmith was finding problems in Frama-C. You might say that Csmith fuzzed itself, but without Frama-C’s deep inspection of the generated programs’ behavior, we couldn’t see the bugs. Recall that Csmith’s key guarantee is that its output is well-defined by the C standard. Frama-C found five bugs where we failed to provide that guarantee.

The bugs found in Frama-C are [listed here](http://j.mp/csmithbugs) and more details can be found in a [short paper](http://www.cs.utah.edu/~regehr/papers/nfm12.pdf) that we (but mainly Pascal) wrote. We hypothesize that random testing would reveal a similar set of issues in other static analysis tools for C. If these tools are being used as part of safety arguments for critical systems, somebody should do this fuzzing.
