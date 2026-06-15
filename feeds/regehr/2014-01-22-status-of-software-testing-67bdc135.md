---
title: Status of Software Testing
url: https://blog.regehr.org/archives/1085
published: "2014-01-22T20:40:34Z"
feed: regehr
guid: http://blog.regehr.org/?p=1085
---

# Status of Software Testing

The other day I received a query from some software engineering researchers who are compiling sort of a survey paper about the status of software testing research; here are the questions, with my answers. I’m interested to hear what the rest of you think about this stuff.

**What do you think are the most significant contributions to testing since 2000, whether from you or from other researchers?**

These would be my choices:

- delta debugging
- symbolic and concolic testcase generation — Klee, SAGE, etc.
- general-purpose open source execution monitoring tools — Valgrind, “clang -fsanitize=undefined”, etc.
- incremental improvements in random testing — QuickCheck and its variants, etc.

**What do you think are the biggest open challenges and opportunities for future research in this area?**

Well, as far as I can tell, gaining confidence in the correctness and security of a piece of software is still both expensive and difficult. Not only that, but this process remains an art. We need to continue making it into a science. Just as a random example, if I want to build a compiler there are about 25 books I can read, all of which cover different aspects of a well-understood body of knowledge. They will tell me the various parts of a compiler and how I should put them together. If I want to become certain that a piece of software does what I hope it does, what books should I read? It’s not even clear. It’s not that there aren’t any good books about testing, but rather that even the good ones fail to contain actionable general-purpose recipes.

Of course not all software is testable. My work on testing GCC has convinced me of this (although I already knew it intellectually). I think there are lots of opportunities in learning how to create testable and/or verifiable software. For example, what could I accomplish if I had a really good concolic tester to help with unit testing? Hopefully, with a fairly low investment I’d end up with good test cases and also good contracts that would be a first step towards verification if I wanted to go in that direction. I think we can take a lesson from CompCert: Xavier didn’t just prove the thing correct, but rather pieced together translation validation and verification depending on which one was appropriate for a given task. Of course we can throw testing into the mix when those other techniques are too difficult. There’s a lot of synergy between these various ways of gaining confidence in software but at present verification and testing are largely separate activities (not least because the verification tools are so hard to use, but that’s a separate topic).

One of the biggest unsolved challenges is finding hidden functionality (whether deliberately or accidentally inserted) in software that operates across a trust boundary. Of course hidden functionality is difficult because the specification gives us few or no clues about where to find it; it’s all about the implementation. There’s no silver bullet for this problem but one idea I like (but haven’t worked on at all) is using continuous mathematics to model the system behavior and then focusing testing effort around discontinuities. [This research](http://www.mpi-sws.org/~rupak/Papers/Symbolic_robustness_analysis.ps) is of the character that I’m trying to describe.
