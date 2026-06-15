---
title: Should Software Evolve?
url: https://blog.regehr.org/archives/383
published: "2011-02-15T19:38:59Z"
feed: regehr
guid: http://blog.regehr.org/?p=383
---

# Should Software Evolve?

Weimer, Nguyen, Le Goues, and Forrest wrote a paper *[Automatically Finding Patches Using Genetic Programming](http://www.cs.virginia.edu/~weimer/p/weimer-icse2009-genprog.pdf)* that proposes and evaluates this approach for fixing a buggy software system:

1. A failure-inducing test case is found
2. The system code is mutated randomly, but using smart heuristics to increase the probability that the part of the program that is broken is modified
3. The new version of the system is tested against the new and old test cases, and is accepted if it passes all of them
4. The fix is minimized using delta debugging

The paper was published in 2009 at ICSE — a highly selective software engineering conference — and it was one of the award papers the year it appeared. In other words, the software engineering community considered it to be one of the best papers of 2009. There is no doubt the approach is interesting and that it should be tried. On the other hand, it seems pretty clear that fixing bugs by genetic programming is sometimes inappropriate. Since the authors of the paper didn’t speculate about when their technique should and shouldn’t be used, I’ll do it.

First off, the method from the Weimer paper is very similar to something I see when teaching CS courses. Some fraction of the students approaches programming problems like this:

1. Write code purporting to solve the assigned problem
2. Run test cases
3. Tweak the logic more or less randomly
4. Go back to 2 until time runs out or all test cases succeed

This is not a good approach. When it works, it permits students to believe they do not need to practice incremental development, and that it’s OK to produce code that would be impossible to maintain. It also defeats the purpose of the assignment — learning — by reducing program construction to a series of mindless tweaks. It works poorly in practice, even for fairly simple assignments: the best outcome is an over-constrained and fragile program that solves some or all test cases, but that contains convoluted logic and needless special cases that render it incorrect for inputs not in the test set. The other outcome (and this happens pretty often) is that the student’s code steadily evolves to be more and more wrong until in the end the code must be thrown away and the student starts over.

Of course, just because it’s a disaster when students use evolutionary programming doesn’t mean that computers shouldn’t do it. My first attempt at a rule for when it’s OK to evolve software was this:

> Software for non-critical applications may be evolved, whereas software for safety-, security-, or mission-critical systems should not be.

But this isn’t a good rule. It’s perfectly fine for certain parts of the software controlling my car’s engine to evolve (I’ll say why below), and some kinds of totally non-critical software should never evolve. Here’s a rule that I think is better:

> Black box software whose correctness can be externally verified can and perhaps should evolve. Program logic that must be understood, modified, analyzed, and (especially) maintained by human developers should not evolve.

For example, if a genetic algorithm is used to evolve the most efficient valve timings for my car engine or the most effective buffer size for my video player, then great — these can be independently verified and the fact that they evolved is irrelevant. On the other hand, if my compiler crashes or emits the wrong code, can it be repaired by an evolutionary technique? No. The logic required to fix any non-trivial compiler bug is intimately entwined with the compiler’s existing logic. Evolving it would be perhaps the dumbest idea on record. My guess is that the core logic of many important software systems (operating systems, virtual machine managers, database engines, etc.) is similarly not amenable to evolutionary bug fixing.

Are evolutionary techniques somehow incompatible with operating systems and compilers? Absolutely not. Parameters to the OS paging or prefetching policy, or the ordering of phases in a compiler, can and should be evolved to be better. But again, this will work because nobody needs to understand the prefetch policy as long as it works. A bad prefetch policy or phase ordering will hurt performance, but won’t break the system.

Of course I’d be happy to be proved wrong, and here’s how the software evolution people can do it: as soon as they ask me to, I’ll start handing them compiler bug reports at the same time I hand these to the LLVM and GCC maintainers. Then, they can try to evolve a fix before the compiler maintainers get around to looking at the problem. This should be easy because evolutionary bug fixing is totally automated whereas manual bug fixing requires stealing time from well-paid and extremely busy developers. Of course the evolved compiler patches should be submitted to the compiler maintainers. If they can get just five evolved patches accepted to the GCC or LLVM trees, I will publicly apologize for being so very, very wrong about their work, and will also buy multiple beers for whatever subset of them I next run into at a conference.

**Update:** I should add that I would very much like evolutionary fixing of compiler bugs to work out, because this method could be combined with my group’s automated detection of compiler bugs. The result would be a compiler that spontaneously becomes more correct.
