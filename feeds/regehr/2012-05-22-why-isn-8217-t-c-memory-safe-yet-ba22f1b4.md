---
title: Why Isn&#8217;t C Memory Safe Yet?
url: https://blog.regehr.org/archives/715
published: "2012-05-22T17:51:44Z"
feed: regehr
guid: http://blog.regehr.org/?p=715
---

# Why Isn&#8217;t C Memory Safe Yet?

Memory safety errors in C/C++ code include null pointer dereferences, out-of-bounds array accesses, use after free, double free, etc. Taken together, these bugs are extremely costly since they are at the root of a substantial fraction of all security vulnerabilities. At the same time, lots of nice research has been done on memory safety solutions for C, some of which appear to not be incredibly expensive in terms of runtime overhead. For example, [baggy bounds checking](http://research.microsoft.com/pubs/101450/baggy-usenix2009.pdf) was shown to reduce Apache throughput by 8% and to increase SPECINT runtime by 60% — both effects are probably of a similar magnitude to compiling at -O0. Here I’m talking about strong versions of memory safety; weaker guarantees from sandboxing can be cheaper.

Although we commonly use memory safety tools for debugging and testing, these are basically never used in production despite the fact that they would eliminate this broad class of security errors. Today’s topic is: Why not? Here are some guesses:

- The aggregate cost of all vulnerabilities due to memory unsafety is less than the cost of a 60% (or whatever) slowdown. Although this is clearly the case sometimes, I don’t buy it always. For example, wouldn’t it be nice to have Acrobat Reader be guaranteed to be free of all vulnerabilities due to memory safety violations, forever, even if it slows down a little? I would take that tradeoff.
- Memory safety solutions come with hidden costs not advertised in the papers such as toolchain headaches, instability, false positives, and higher overheads on real workloads. My guess is that these costs are real, but that people may also exaggerate them because we don’t like toolchain changes.
- People are not very good at estimating the aggregate costs that will be incurred by not-yet-discovered vulnerabilities. My guess is that this effect is also real.

Anyway, I’m interested by this kind of apparent paradox where a technical problem seems solvable but the solutions aren’t used. There are lots of other examples, such as integer overflows. This seems related to Worse is Better.
