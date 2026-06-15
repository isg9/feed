---
title: Academic Attention for Undefined Behavior
url: https://blog.regehr.org/archives/746
published: "2012-07-09T03:08:28Z"
feed: regehr
guid: http://blog.regehr.org/?p=746
---

# Academic Attention for Undefined Behavior

*Undefined behaviors* are like blind spots in a programming language; they are areas where the specification imposes no requirements. In other words, if you write code that executes an operation whose behavior is undefined, the language implementation can do anything it likes. In practice, a few specific undefined behaviors in C and C++ (buffer overflows and integer overflows, mainly) have caused, and are continuing to cause, a large amount of economic damage in the form of exploitable vulnerabilities. On the other hand, undefined behaviors have advantages: they simplify compiler implementations and permit more efficient code to be generated. Although the stakes are high, no solid understanding of the trade-offs exists because, for reasons I don’t understand, the academic programming languages community has basically ignored the issue. This may be starting to change, and recently I’ve learned about two new papers about undefined behavior, [one from UIUC](https://www.ideals.illinois.edu/handle/2142/30780) and the other (not yet publicly available, but hopefully soon) from MIT will appear in the [“Correctness” session at APSYS 2012](http://apsys2012.kaist.ac.kr/program/) later this month. Just to be clear: plenty has been written about avoiding specific undefined behaviors, generally by enforcing memory safety or similar. But prior to these two papers, nothing has been written about undefined behavior in general.

**UPDATE:** I should have mentioned Michael Norrish’s work. Perhaps [this paper](http://www.cl.cam.ac.uk/~mn200/PhD/esop1999.pdf) is the best place to start. [Michael’s thesis](http://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-453.pdf) is also excellent.

**UPDATE:** The MIT paper is [available now](http://people.csail.mit.edu/nickolai/papers/wang-undef.pdf).
