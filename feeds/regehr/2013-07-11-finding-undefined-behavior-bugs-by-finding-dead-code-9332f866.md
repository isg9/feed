---
title: Finding Undefined Behavior Bugs by Finding Dead Code
url: https://blog.regehr.org/archives/970
published: "2013-07-11T02:52:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=970
---

# Finding Undefined Behavior Bugs by Finding Dead Code

Here’s a [draft of a very cool paper](http://pdos.csail.mit.edu/papers/stack:sosp13.pdf) by [Xi Wang](http://pdos.csail.mit.edu/~xi/) and others at MIT; it is going to appear at the [next SOSP](http://sigops.org/sosp/sosp13/).

The idea is to look for code that becomes dead when a C/C++ compiler is smart about exploiting undefined behavior. The classic example of this class of error was found in the Linux kernel several years ago. The code was basically:

```
struct foo *s = ...;
int x = s->f;
if (!s) return ERROR;
... use s ...

```

The problem is that the dereference of `s` in line 2 permits a compiler to infer that `s` is not null (if the pointer is null then the function is undefined; the compiler can simply ignore this case). Thus, the null check in line 3 gets silently optimized away and now the kernel contains an exploitable bug if an attacker can find a way to invoke this code with a null pointer. [Here’s another example that I like](https://code.google.com/p/nativeclient/issues/detail?id=245) where the compiler silently discarded a security check that invoked undefined behavior. There are more examples in the paper and I encourage people to look at them. The point is that these problems are real, and they’re nasty because the problem can only be seen by looking at the compiler’s output. Compilers are getting smarter all the time, causing code that previously worked to break. A sufficiently advanced compiler is indistinguishable from an adversary.

So the premise of this paper is that if you write some code that executes conditionally, and that code can be eliminated by a compiler that understands how to exploit undefined behavior, then there’s a potential bug that you’d like to learn about. But how can we find this kind of code? One way would be to instrument the compiler optimization passes that eliminate code based on undefined behavior. This has some problems. First, due to complex interactions between optimization passes in a real compiler, it is not at all straightforward to determine whether a particular piece of dead code is that way due to undefined behavior, vs. being dead in a boring way. [Chris Lattner’s article about undefined behavior](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_14.html) has some good discussion of this issue. The second problem is that if we, for example, instrument LLVM to detect cases where it optimizes away code due to undefined behavior, this doesn’t tell us much about problems that we might run into when using a different compiler, or using the next version of LLVM. In particular, it would be nice to be able to find “time bombs” — undefined behavior bugs that aren’t yet exploited by a compiler, but that might be exploited in the future.

The approach taken in this paper was to develop a custom undefined behavior detector based on static analysis of a single function at a time. It takes LLVM IR and converts it into a [SAT instance](http://en.wikipedia.org/wiki/Boolean_satisfiability_problem) that is satisfiable when code can be eliminated. This has a couple of advantages. First, by considering one function at a time, the tool can scale to very large programs (note that a single function at the LLVM level may contain inlined code from other functions — this increases precision without slowing things down too much). Second, modern SAT solvers are quite strong, giving the tool the opportunity to find bugs that cannot yet be exploited by production compilers. Third, by writing a standalone tool, problems of being entangled with all of the incidental complexity that is found in a collection of real compiler passes are avoided.

At this point you are probably thinking: “I do not want some stupid tool telling me about every piece of dead code in a huge legacy application, because there are millions of those and almost none of them are bugs.” The solution has two parts. First, the most obvious dead code gets eliminated by running LLVM passes like [mem2reg](http://llvm.org/docs/Passes.html#passes-mem2reg), [SCCP](http://llvm.org/docs/Passes.html#passes-sccp), and [DCE](http://llvm.org/docs/Passes.html#passes-dce). Second, the authors have a clever solution where they run their tool twice: first without knowledge of undefined behavior, and second with it. A bug is only reported if the second pass can eliminate code that the first one cannot. Based on this technique, the paper reports a very low rate of false positives.

One thing that is interesting about the tool in this paper is that it has very conservative criteria for reporting a bug: either undefined behavior is invoked on all paths (e.g. it finds a [type 3 function](https://blog.regehr.org/archives/213)) or else it finds code that is made dead by exploiting undefined behavior. Although this conservative behavior could be seen as a disadvantage, in many ways it is a big advantage because the problems that it flags are probably serious, because the code that gets eliminated by the compiler is often a safety check of some sort. The authors were able to find, report, and convince developers to fix 157 bugs in open source programs, which is impressive. They also provide some good anecdotes about developers who could not be convinced to fix undefined behavior bugs, something I’ve also run into.

In summary, by adopting a solid premise (“developers want to know when code they write can be eliminated based on exploitation of undefined behavior”) the authors of this paper have found a way to home in on a serious class of bugs and also to avoid the false positive problems that might otherwise plague an intraprocedural static analysis tool.

**UPDATE from August 2013:** [STACK is available now](http://css.csail.mit.edu/stack/).
