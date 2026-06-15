---
title: A New Compiler Fuzzing Paper
url: https://blog.regehr.org/archives/1061
published: "2013-11-12T22:05:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=1061
---

# A New Compiler Fuzzing Paper

Google Scholar’s recommendation engine has turned out to be a great resource for learning about new papers. Recently [this paper](http://ist.ksc.kwansei.ac.jp/~ishiura/publications/C2013-10.pdf) about compiler fuzzing turned up in my feed and I was hooked after noticing that they claim to find a lot more compiler bugs during a 12-hour fuzzing run than Csmith can find.

The work in the paper succeeds by addressing a problem that we punted on while developing Csmith: generating expressions that are statically guaranteed to be free of integer undefined behaviors. They accomplish this using static analysis combined with random search. Csmith, in contrast, wraps all math operations with safety checks in order to prevent undefined behaviors dynamically. This is a klunky solution that — as the authors of the new paper note — reduces Csmith’s expressiveness.

The character of the bugs they found follows directly from the generation strategy:

- [GCC PR 56250](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=56250)
- [GCC PR 56899](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=56899)
- [GCC PR 56984](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=56984)
- [GCC PR 57083](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57083)
- [GCC PR 57131](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57131)
- [GCC PR 57656](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57656)
- [GCC PR 57829](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57829)
- [GCC PR 58088](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=58088)
- [LLVM PR 15607](http://llvm.org/bugs/show_bug.cgi?id=15607)
- [LLVM PR 15940](http://llvm.org/bugs/show_bug.cgi?id=15940)
- [LLVM PR 15941](http://llvm.org/bugs/show_bug.cgi?id=15941)
- [LLVM PR 15959](http://llvm.org/bugs/show_bug.cgi?id=15959)
- [LLVM PR 16108](http://llvm.org/bugs/show_bug.cgi?id=16108)

These bugs are, frankly, a little surprising. GCC and LLVM are mature tools and I’d have hoped they were beyond these simple math errors.

Besides the obvious fact that this new tool is generating expressions that Csmith cannot generate, there’s also a more subtle thing going on, which is that any new C code generator is likely to find bugs that Csmith cannot. This is because there are a hundred little choices that get made while generating test cases such as choice of identifiers, choice of function size, how many parentheses to put in and where, etc. Many of these have no effect on bug-finding power but some do, and there’s really no way to know which of them matter in advance. This is one of the most dissatisfying things about random testing.

As far as I can tell, the new compiler testing tool has not been released. Hopefully the authors will release it. If they do release it under an appropriate license, one thing I would consider doing is integrating it into Csmith. This would be easy to do by having Csmith just fail to generate code for a few functions and then generate them using this new tool. At that point we should get the benefits of both tools.

**UPDATE:** Here are two more C compiler fuzzers I just learned about:

- [ccg](https://github.com/Merkil/ccg) — can only find crash bugs, but has a [very impressive record](https://github.com/Merkil/ccg/blob/master/reported_bugs) already

- [icFuzz](http://thread.gmane.org/gmane.comp.compilers.llvm.devel/63269) — unfortunately this is an internal Intel tool
