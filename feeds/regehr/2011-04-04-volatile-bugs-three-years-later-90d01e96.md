---
title: Volatile Bugs, Three Years Later
url: https://blog.regehr.org/archives/503
published: "2011-04-04T04:21:53Z"
feed: regehr
guid: http://blog.regehr.org/?p=503
---

# Volatile Bugs, Three Years Later

Almost exactly three years ago Eric Eide and I submitted a paper [Volatiles Are Miscompiled, and What to Do about It](http://www.cs.utah.edu/%7Eregehr/papers/emsoft08-preprint.pdf) to the 8th International Conference on Embedded Software (EMSOFT 2008). The points made in this paper were that:

- C compilers fail to reliably translate accesses to volatile-qualified objects
- we can automatically detect these failures
- we can (at least in some cases) automatically work around these failures

Gratifyingly, this paper found an audience among embedded software practitioners and in 2010 it was one of the most highly-downloaded PDFs on the Utah CS web server. Not gratifyingly, it isn’t clear that compilers are, in general, much better at volatile than they were three years ago (we haven’t quantified this, it’s just my general feeling). The root of the problem is that there is a tension between volatile and optimizations: it’s hard to make fast code that is still volatile-correct.

The motivation for writing this paper originated in a lecture for my Advanced Embedded Systems course that I gave in 2006 or 2007. I was presenting some fragments of C code along with their translations into assembly, in order to illustrate the effect of the volatile qualifier, when a student raised his hand and said that one of the translations was incorrect. I assumed that I had made a cut-and-paste error and moved on with the lecture. However, when I checked up later, it turned out there was no cut and paste error: the compiler had been wrong (this was CodeWarrior for ColdFire, I believe). This was surprising, so I kept playing around and the situation got worse every time I tried a new compiler or wrote a new code fragment. Eventually it became clear that systematic wrongness existed and I needed to write something up, though I had no idea how to turn this into any kind of respectable academic paper. Eric saved the day by taking Randprog, an existing random C program generator, and extending it to generate code using volatile. Also, he hacked Valgrind to count accesses to volatiles, giving us the automatic detector. Finally, my student Nathan hacked up a CIL pass for turning volatile accesses into calls to helper functions (I don’t recall why we didn’t include Nathan as an author on the paper — we probably should have). At this point we had a paper. I like this story because it illustrates the way systems research often works in practice. It does not proceed in a nice chain from hypothesis to solution to evaluation. Rather, it begins with a niggling suspicion that something is wrong, and proceeds in little fits and starts, across plenty of dead ends, until finally it becomes clear that a useful result has emerged.

By far the most interesting development in volatile-land during the last three years is CompCert, which has a provably correct implementation of volatile. This is, as I’ve [said here before](https://blog.regehr.org/archives/478), and will no doubt keep saying, a very impressive result.

The volatile qualifier is a legitimate solution to a real problem in early C compilers: they would optimize away critical accesses to hardware device registers. Prior to the introduction of volatile, extremely dodgy hacks were used to avoid miscompilation. However, in retrospect, I believe volatile has proved more trouble than it’s worth, and that C/C++ would be better off without it. The alternative is to use an explicit function call to access variables that live in special kinds of memory; these calls need not have high overhead since they can be inlined. The argument for explicit accesses comes not just from the compiler side, but also from the user side. Linus Torvalds has [ranted about this](http://kerneltrap.org/Linux/Volatile_Performance), and he’s right.

I suspect that Eric and I need to write at least one more volatile paper during the next year or two. Some things that have changed:

- CompCert supports volatile, so it is available as a basis for comparison
- GCC and LLVM are less prone to non-volatile miscompilations than they used to be, making it much easier for us to assess the reliability of automatically turning volatile accesses into calls to helper functions
- my student Yang Chen has created a new Pin-based volatile bug detector that works better than the Valgrind-based one
- the Pin tool supports testing whether the generated code correctly copes with the case where the volatile location returns a “fresh” value each time it is read — we never tested this before
- Csmith + the Pin tool support testing whether accesses to volatile locations are illegally reordered, which we never tested before

It will be fun to see what kinds of bugs are turned up by the far more aggressive testing we can now do, compared to what we could do in 2008. However, the more interesting thing will be to implement the automatic volatile bug workaround in GCC and LLVM, by turning volatile accesses into calls to helper functions in the frontends, and turning them back into memory accesses in the backends. This should achieve near total volatile correctness and will also permit hundreds of silly special cases to be removed from these compilers’ optimizers. Ideally the compiler developers will adopt this approach, though I suspect this depends on the performance of the generated code (it should be decent, but won’t be as good as the current, broken approach).
