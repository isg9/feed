---
title: Design and Evolution of C-Reduce (Part 2)
url: https://blog.regehr.org/archives/1679
published: "2019-08-12T15:20:11Z"
feed: regehr
guid: http://blog.regehr.org/?p=1679
---

# Design and Evolution of C-Reduce (Part 2)

[Part 1 of this series](https://blog.regehr.org/archives/1678) introduced C-Reduce and showed how it combines a domain-independent core with a large collection of domain-specific passes in order to create a highly effective test-case reducer for C and C++ code. This part tells the rest of the story and concludes.

## Parallel Test-Case Reduction

C-Reduce’s second research contribution is to perform test-case reduction in parallel using multiple cores. The parallelization strategy, based on the observation that most variants are uninteresting, is to speculate along the uninteresting branch of the search tree. Whenever C-Reduce discovers an interesting variant, all outstanding interestingness tests are killed and a new line of speculation is launched. This is the same approach that was subsequently rediscovered by the creator of [halfempty](https://github.com/googleprojectzero/halfempty).

C-Reduce has a policy choice between taking the first interesting variant returned by any CPU, which provides a bit of extra speedup but makes parallel reduction non-deterministic, or only taking an interesting variant when all interestingness tests earlier in the search tree have reported that their variants are uninteresting. We tested both alternatives and found that the speedup due to non-deterministic choice of variants was minor. Therefore, C-Reduce currently employs the second option, which always follows the same path through the search tree that non-parallel C-Reduce would take. The observed speedup due to parallel C-Reduce is variable, and is highest when the interestingness test is relatively slow. Speedups of 2-3x vs. sequential reduction are common in practice.

## Applicability and Moving Towards Domain-Independence

Something that surprised us is how broadly applicable C-Reduce is to test cases in languages other than C and C++. Our users have found it effective in reducing Java, Rust, Julia, Haskell, Racket, Python, SMT-LIB, and a number of other languages. When used in this mode, the highly C/C++-specific C-Reduce passes provide no benefit, but since they fail quickly they don’t do much harm. (C-Reduce also has a `--not-c` command line option that avoids running these passes in the first place.)

One might ask why C-Reduce is able to produce good results for languages other than C and C++; the answer appears to be based on the common structural elements found across programming languages in the Algol and Lisp families. In contrast, in our limited experience, C-Reduce does a very poor job reducing test cases in binary formats such as PDF and JPEG. Other reducers, such as the afl-tmin tool that ships with afl-fuzz, work well for binary test cases.

A consequence of the modular structure of C-Reduce is that while its transformation passes are aimed at reducing C and C++ code, the C-Reduce core is completely domain-independent. C-Reduce has been forked to create a [highly effective reducer for OpenCL programs](https://github.com/mpflanzer/clreduce). We believe it would be relatively straightforward to do something similar for almost any other programming language simply by tailoring the passes that C-Reduce uses.

## Limitations

When used for its intended purpose, when does C-Reduce work badly? We have seen two main classes of failure. First, C-Reduce can be annoyingly slow. This typically happens when the passes early in the phase ordering, which are intended to remove a lot of code quickly, fail to do this. Second, highly templated C++ sometimes forces C-Reduce to terminate with an unacceptably large (say, >1 KB) final result. Of course this is better than nothing, and subsequent manual reduction is usually not too difficult, but it is frustrating to have written 69 different Clang-based passes and to still find effectively irreducible elements in test cases. The only solution — as far as we know — is to strengthen our existing transformation passes and to create more such passes.

A small minority of compiler bugs appears to be impossible to trigger using a small test case. These bugs are exceptions to the [small scope hypothesis](http://projects.csail.mit.edu/mulsaw/papers/SSH.ps). They typically stem from resource-full bugs in the compiler. For example, a bug in register spilling code requires the test case to use enough registers that spilling is triggered; a bug in logic for emitting long-offset jumps requires the test case to contain enough code that a long offset is required; etc. These test cases are just difficult to work with, and it is not clear to us that there’s anything we can do to make it easier to debug the issues that they trigger.

## C-Reduce Design Principles

In summary, C-Reduce was designed and implemented according to the following principles:

1. Be aggressive: make the final reduced test case as small as possible.

2. Make the reducer fast, for example using parallelism, careful phase ordering of passes, and avoiding unnecessary I/O traffic, when this can be done without compromising the quality of the final output.

3. Make it as easy as possible to implement new passes, so that many domain-specific passes can be created.

4. Keep the C-Reduce core domain-independent.

5. Focus only on reduction, delegating all other criteria to the user-supplied interestingness test.

## Directions for Future Test-Case Reduction Research

Although perhaps a few dozen papers have been written about test-case reduction since Hildebrandt and Zeller’s initial paper 19 years ago, I believe that this area is under-studied relative to its practical importance. I’ll wrap up this article with a collection of research questions suggested by our experience in over a decade of work on creating a highly aggressive reducer for C and C++.

**What is the role of domain-specificity in test case reduction?** Researchers who cite C-Reduce appear to enjoy pointing out that it is very domain-specific (nobody seems to notice that the C-Reduce core is domain-independent, and that the pass schedule is easy to modify). The implication is that domain-specific hacks are undesirable and, of course, an argument against such hacks would be forceful if backed up by a test-case reducer that produced smaller final C and C++ code than C-Reduce does, without using domain knowledge. So far, such an argument has not been made.

The open research question is whether domain knowledge is actually necessary, or if a domain-independent test-case reducer can beat C-Reduce at its own game. The most impressive such effort that we are aware of is David MacIver’s [structureshrink](https://github.com/DRMacIver/structureshrink), which uses relatively expensive search techniques to infer structural elements of test cases that can be used to create variants. Anecdotally, we have seen structureshrink produce reduced versions of C++ files that are smaller than we would have guessed was possible without using domain knowledge.

**What is the role of non-greedy search in test-case reduction?** In many cases, the order in which C-Reduce runs its passes has little or no effect on the final test case. In other words, the search is often diamond-shaped, terminating at the same point regardless of the path taken through the search space. On the other hand, this is not always the case, and when the search is not diamond-shaped, a greedy algorithm like C-Reduce’s risks getting stopped at a local minimum that is worse than some other, reachable minimum. The research question is how to get the benefit of non-greedy search algorithms without making test-case reduction too much slower.

**What other parallelization methods are there?** C-Reduce’s parallelization strategy is pretty simple, gives a modest speedup in practice, and always returns the same result as sequential reduction. There must be other parallel test-case reduction strategies that hit other useful points in the design space. This is, of course, related to the previous research question. That is, if certain branchings in the search tree can be identified as being worth exploring in both directions, this could be done on separate cores or machines.

**What is the role of canonicalization in test-case reduction?** A perfectly canonicalizing reducer — that reduces every program triggering a given bug to the same reduced test case — would be extremely useful. Although it seems impossible to achieve this in all cases, there are many relatively simple strategies that can be employed to increase the degree of canonicalization, such as putting arithmetic expressions into a canonical form, assigning canonical names to identifiers, etc. C-Reduce has a number of transformations that are aimed at canonicalization rather than reduction. For example, the reduced test case at the top of part 1 of this piece has four variables a, b, c, and d, which appear in that order. I believe that more work in this direction would be useful.

**Can we avoid bug hijacking?** Test reduction sometimes goes awry when the bug targeted by the reduction is “hijacked” by a different bug. In other words, the reduced test case triggers a different bug than the one triggered by the original. During a compiler fuzzing campaign this doesn’t matter much since one bug is as good as another, but hijacking can be a problem when the original bug is, for example, blocking compilation of an application of interest. Hijacking is particularly common when the interestingness test is looking for a non-specific behavior such as a null pointer dereference. C-Reduce pushes the problem of avoiding hijacking onto the user who can, for example, add code to the interestingness test looking for specific elements in a stack trace. The research question here is whether there are better, more automated ways to prevent bug hijacking.

## Obtaining C-Reduce

Binary C-Reduce packages are available as part of software distributions including Ubuntu, Fedora, FreeBSD, OpenBSD, MacPorts, and Homebrew. [Source code can be found here](https://github.com/csmith-project/creduce).

## Acknowledgments

C-Reduce was initially my own project, but by lines of code the largest contributor by a factor of two is my former student Yang Chen, now a software engineer at Microsoft. Yang wrote effectively all of the Clang-based source-to-source transformation code, more than 50,000 lines in total. Eric Eide, a research professor in computer science at the University of Utah, is the other major C-Reduce contributor (and gave useful feedback on a draft of this piece). Our colleagues Pascal Cuoq, Chucky Ellison, and Xuejun Yang also contributed to the project, and we have gratefully received patches from a number of external contributors. Someone created [a fun visualization of the part of C-Reduce’s history that happened on Github](https://www.youtube.com/watch?v=2NSX5Gr_bYo).
