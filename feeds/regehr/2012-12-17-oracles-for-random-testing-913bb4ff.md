---
title: Oracles for Random Testing
url: https://blog.regehr.org/archives/856
published: "2012-12-17T20:06:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=856
---

# Oracles for Random Testing

Random testing is a powerful way to find bugs in software systems. However, to actually find a bug it’s necessary to be able to automatically tell the difference between a correct and an incorrect execution of the system being tested. Sometimes this is easy: we’re just looking for crashes. On the other hand, there are lots of incorrect executions that don’t crash and we’d like to find those too. A *test oracle* is what we call a way to automatically find erroneous executions. This post is intended to be a public brainstorm: I’ll list all the kinds of oracles I’ve thought of and hopefully you readers will leave comments filling in the gaps.

**Programming environment rule violations.** Most programming environments have some amount of built-in checking for rule violations. For example, the Linux execution environment will crash a process that attempts to access an unmapped part of the address space. Java has lots of reasons to throw exceptions. In many cases, the execution environment can be asked or forced to act as a more effective oracle. For example, Perl code can be run under “use warnings” and the Linux environment can enforce a dramatically expanded set of rules by running a program under Valgrind. Some libraries that you use can be compiled with extra input validation turned on. Also, the code being tested has to cooperate by failing to hide rule violations. For example, if my Linux program masks all signals or my Java program catches and ignores all exceptions, these oracles are going to have a hard time revealing anything interesting.

**Function/inverse pairs.** If we can find a pair of operations that undo each other, we can generate a random test case, apply the function to it, apply the inverse function to the result, and compare the output with the original test case. I [wrote about this earlier](https://blog.regehr.org/archives/708).

**Nops.** Many codes have semantic nops that can be exploited to find bugs (a function/inverse pair is a kind of nop but it seemed important enough to split out into its own category). For example, if we’re testing a transaction processing system we might begin a lot of transactions, force them all to abort, and then check that the data has not been changed. If we’re testing a JVM we can force the garbage collector to execute after every instruction and then check that the program’s output is not changed. At the hardware level, flushing the pipeline or caches should not affect the ongoing computation. Concurrent software (usually) should not change its result if we put random sleeps inside critical sections or force the thread scheduler to perform many more context switches than it usually would. A system that is designed to recover from transient faults, such as dropped or corrupted network messages, should not change its behavior when such a fault is artificially induced.

**Nullspace transformations.** If I’m testing a compiler, I might take some test case and replace every occurrence of x with (1+x-1) or (1.0 \* x) or perhaps a call to a silly getter or setter function. The transformed program, when compiled, should behave the same as the original, as long as I’m super careful not to run afoul of overflows and such. If I’m testing a spellchecker, I might introduce whitespace changes or additional correctly-spelled words and make sure its behavior is not changed. The basic idea is that even if we can’t predict the behavior of a system, we can probably think of ways to change inputs to that system in such a way that the output of a correct system will not change. I’m grabbing the name of this kind of oracle from [this paper](http://www.cs.indiana.edu/ftp/techreports/TR564.pdf).

**Multiple versions.** Occasionally we have multiple independent implementations of the same specification, such as a collection of Java compilers, and they can be tested against each other. Even when this is not the case, we can often test today’s version of our system against yesterday’s version or perhaps the previous release. Some kinds of program, such as optimizing compilers, effectively contain many versions in one: the -O0 version of GCC, the -O1 version, the -O2 version, etc.

**Assertions.** Internal consistency checks can be a very powerful kind of oracle. Here, we run our code on a random input and simply wait for the code to report an error.

**Resource limits.** Sometimes we can be pretty sure that our code should not, for example, use more than 1 GB of RAM or 1 minute  of CPU time on simple random inputs. In that case, we can use a call such as ulimit to arrange for the OS to signal an error when the resource bound is exceeded. Finding good resource limits can be hard.
