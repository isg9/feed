---
title: Squeezing a CS Research Idea
url: https://blog.regehr.org/archives/537
published: "2011-05-24T22:13:19Z"
feed: regehr
guid: http://blog.regehr.org/?p=537
---

# Squeezing a CS Research Idea

*\[Disclaimer: I wrote about this topic [once before](https://blog.regehr.org/archives/6), but it was buried in a longer piece. Also, it was in one of the first posts on this blog and I don’t think anyone read it. **Update**: I checked the stats. Nobody read it.\]*

A hammer I like to use when reviewing papers and PhD proposals is one that (lacking a good name) I call the “squeeze technique” and it applies to research that optimizes something. To squeeze an idea you ask:

1. How much of the benefit can be attained without the new idea?
2. If the new idea succeeds wildly, how much benefit can be attained?
3. How large is the gap between these two?

Though this is simple, surprisingly often people have failed to work it out for themselves. Let’s go through an example and then get back to the details.

In 2004 Markus Mock wrote [a clever paper](http://www.cs.pitt.edu/%7Emock/papers/clei2004.pdf) which squeezes the idea of programmer specified aliasing, as specified by the C99 language. The “restrict” qualifier added in that version of the language serves as a hint from developers to the compiler; it means that the qualified pointer will not be aliased by other pointers, permitting better code generation in some situations.

Mock’s idea was to instrument a collection of benchmarks to record actual pointer behavior and then use this information to add the maximum possible number of “restrict” qualifiers. In other words, he used dynamic information to optimistically add a restrict qualifier in every location where this was permissible for the inputs used during benchmarking. He then re-ran the modified benchmarks and found less than 1% average speedup.

This result means that for the given compilers and benchmarks, no conceivable human intervention or static pointer analysis can achieve more than 1% average speedup through the addition of restrict qualifiers. The conclusion was that since “restrict” is dangerous in practice (misplaced qualifiers will cause incorrect code to be generated) programmer-specified aliasing is not generally a good idea.

Squeezing a research idea has three elements. The first — left implicit in the list above — is to select a metric. In general this should be externally meaningful: something that people other than computer scientists would care about, perhaps because it has economic consequences. For example speedup, throughput, MTTF, or energy consumption might make a good metric (when coupled with good workloads, a good test platform, good experiment design, etc.). In contrast, the number of “restrict” qualifiers, the average size of an alias set, and the number of context switches are probably poor top-level metrics because they enjoy (at best) an indirect relationship with something a person on the street might be worried about.

The second important choice is the baseline: the amount of benefit achieved without the current idea. This can be tricky and there are many ways of going wrong (probably baseline selection deserves its own blog post). A pernicious mistake is to choose a straw man: an obviously primitive or defective baseline. For example, maybe I’ve created a highly parallelizable algorithm for coalescing hoovulators. Then, I choose the single-processor version of my algorithm as the baseline and go on to show that I can achieve near-linear scalability up to 512 processors. This is a great result unless I am ignoring Stickleback’s 1983 algorithm that is non-parallelizable but performs the same job 750 times faster than my single-processor version, on the same hardware.

The third important element of the squeeze technique is determining an upper bound on improvement. In many cases — as Mock showed — a little creativity is all that is required. Other examples:

- Amdahl’s Law bounds parallel speedup
- The power consumption of the disk, display, and memory often bound the increase in mobile device lifetime that can be attained by reducing processor power usage
- Time spent copying data can bound throughput improvements in networking
- Time spent traveling at 0.3c can bound latency improvements in networking
- CPU, memory, or I/O saturation can bound improvements in throughput due to better resource scheduling

In summary, no matter what we are trying to optimize, there is probably a component that we can improve and a component that we cannot. Understanding the relative magnitudes of these components is a powerful tool for helping to select a research problem to work on. I find myself extremely reluctant to sign off on a document like a PhD proposal that does not display evidence of this sort of thinking.
