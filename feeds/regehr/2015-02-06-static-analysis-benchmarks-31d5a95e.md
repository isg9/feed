---
title: Static Analysis Benchmarks
url: https://blog.regehr.org/archives/1217
published: "2015-02-06T17:11:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=1217
---

# Static Analysis Benchmarks

Many programmers would agree that static analysis is pretty awesome: it can find code defects that are very hard to find using testing and walkthroughs. On the other hand, some scientific validation of the effectiveness of static analysis would be useful. For example, [this nice 2004 paper](https://www.ll.mit.edu/mission/communications/cyber/CSTcorpora/04_TestingStatic_Zitser.pdf) found that when five analyzers were turned loose against some C programs containing buffer overflows, only one of them was able to beat random guessing in a statistically significant fashion.

More recently, in 2014, some researchers at the [Toyota InfoTechnology Center](http://www.us.toyota-itc.com/) published a paper evaluating six static analyzers using a synthetic benchmark suite containing more than 400 pairs of C functions, where each pair contains one function exemplifying a defect that should be found using static analysis and the other function, while looking fairly similar, does not contain the defect. The idea is that a perfect tool will always find the bug and not find the not-bug, if you see what I mean. The results show that different tools have different strengths; for example, CodeSonar dominates in detection of concurrency bugs but PolySpace and QA-C do very well in finding numerical bugs. With a few exceptions, the tools do a good job in avoiding false positives. At this point I feel like I have to refer [Coverity’s classic writeup](http://cacm.acm.org/magazines/2010/2/69354-a-few-billion-lines-of-code-later/fulltext) about this kind of thing just because it’s awesome.

Shin’ichi Shiraishi, the lead author of the Toyota paper, contacted me asking if I’d help them release their benchmarks as open source, and I have done this: the [repository is here](https://github.com/regehr/itc-benchmarks), and [here is a brief document describing the benchmarks](https://github.com/regehr/itc-benchmarks/blob/master/docs/spec.pdf). (I just want to make it clear that while I own the Github repo, the work here was entirely done by Shin’ichi and his colleagues — I added the simple autotooling, fixed a few portability problems in the benchmarks such as using intptr\_t for integers that will be cast to and from pointer values, and did some other lightweight cleanup.) Anyway, I am very happy that Shin’ichi and his organization were able to release these benchmarks and we hope that they will be useful to the community.

**Update from 2/25/15:** Whoops — it looks like Coverity/Synopsys has a [DeWitt clause](http://sqlmag.com/sql-server/devils-dewitt-clause) in their EULA and based on this, they sent me a cease and desist notice. In accordance with their request, I have removed Shin’ichi’s group’s paper from the Github repo and also removed the link to that paper from this piece.
