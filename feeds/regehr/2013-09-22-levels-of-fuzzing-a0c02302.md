---
title: Levels of Fuzzing
url: https://blog.regehr.org/archives/1039
published: "2013-09-22T23:10:56Z"
feed: regehr
guid: http://blog.regehr.org/?p=1039
---

# Levels of Fuzzing

[Differential Testing for Software](http://www.cs.dartmouth.edu/~mckeeman/references/DifferentialTestingForSoftware.pdf) is one of my favorite papers about software testing: it is one of the few pre-delta-debugging papers to describe automated test-case reduction and it contains many pieces of wisdom about software testing in general and random testing in particular. One of them outlines a hierarchy of test case generation strategies:

> For a C compiler, we constructed sample C source files at several levels of increasing quality:
> 1. Sequence of ASCII characters
>
> 2. Sequence of words, separators, and white space
>
> 3. Syntactically correct C program
>
> 4. Type-correct C program
>
> 5. Statically conforming C program
>
> 6. Dynamically conforming C program
>
> 7. Model-conforming C program

The paper goes on to describe how test cases at different levels reveal different kinds of flaws in a compiler. McKeeman’s choice of the words “increasing quality” is interesting because the set of test cases that can be generated at each level is a strict subset of the set of test cases generated at the previous level. In other words, for example, every type-correct C program is syntactically correct but the converse is not true. Since the expressiveness of the test-case generator decreases with increasing level, so must the size of the set of bugs that can be triggered.

So where does “increasing quality” come from? The problem is that in the real world, testing budgets are finite. Therefore, while a random sequence of ASCII characters can in principle trigger every single compiler bug, in practice we would have to run a very large number of test cases before finding the kind of bug-triggering programs [shown here](https://blog.regehr.org/archives/1036). Actually, it is not necessarily the case that every compiler bug can be triggered by 7-bit ASCII codes. To be safe, we should probably define test level 0: a random bit stream.

Given the low probability of triggering difficult optimizer and backend bugs using random bits or characters, we might interpret “increasing quality” as meaning “increasing coverage of compiler code within a reasonable test budget.” So we have this interesting situation where decreasing the set of inputs that we are willing to generate increases the expected level of coverage of the system under test.

Although McKeeman reports finding bugs at levels 1-5, for the Csmith work we did not bother with any of levels 0-5. It’s not that we didn’t think test cases at these levels would trigger some bugs, but rather that we were mainly interested in sophisticated optimizer bugs that are very hard to spot until level 6. My guess, however, is that a high-quality modern compiler (GCC, Clang, Intel CC, MSVC, etc.) is probably not going to have a lot of bugs that you can find using levels 0-3.

Bart Miller’s [original fuzzer](ftp://ftp.cs.wisc.edu/paradyn/fuzz/fuzz-original.tar.gz) operates only at level 1. The fact that it found a lot of bugs tells us that the programs being tested contained a lot of bugs in the code that can be reached using level 1 inputs within a very small (by today’s standards) test budget. Basically, the then-unfuzzed UNIX utilities worked in the common case but were very fragile with respect to non-standard inputs. Also, it is most likely the case that many of these programs lacked the kind of sophisticated constraints on what constitutes a valid input that C compilers have.

Miller’s fuzzer, McKeeman’s fuzzer, and Csmith are all *generational fuzzers*: they generate test cases from scratch. In the time since the original fuzzing paper, the term “fuzz” has come to often mean *mutational fuzzing* where a valid input to the system under test (such as a PDF document, an Excel spreadsheet, or whatever) is randomly modified. This kind of fuzzer does not fit neatly into McKeeman’s hierarchy, it can be seen as a hybrid of a level 0 fuzzer and a level 7 fuzzer.

I want to finish up by making a few points. First, if you are writing a generational fuzzer, it is worth thinking about which level or levels that fuzzer should operate at. In other words, McKeeman’s hierarchy is not just a way to look at compiler fuzzers, it’s a way to understand all generational fuzzers. I also think it is useful as a means of understanding mutational fuzzers even if they do not fit into the hierarchy so neatly. For example, one can imagine a mutational fuzzer that instead of modifying test cases at level 0, modifies them at level 3 or 4. My second point is that a comprehensive fuzzing campaign should operate at multiple levels of McKeeman’s hierarchy. In other words, it would be useful to create a family of fuzzers, for example one operating at each of levels 0-7, and to split up the testing budget among them. The details of how this split is performed are probably not that important, but in practice I would look at the bug yield of each level after a couple of weeks of fuzzing and then use that information to increase the budget allocated to the more productive levels. However, if I really cared about the quality of the system under test, under no circumstances would I allocate zero test budget to any of the fuzzers. As I said earlier, for the Csmith work we ignored all but level 6: this was because we had a specific research agenda, as opposed to wanting to maximize the quality of some compiler. Finally, it is not the case that all of McKeeman’s levels make sense for all systems under test. For example, if we are fuzzing an API instead of a standalone tool, it probably makes no sense to pass random bit streams to the API.
