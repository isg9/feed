---
title: C and C++ Aren&#8217;t Future Proof
url: https://blog.regehr.org/archives/880
published: "2013-01-20T22:17:47Z"
feed: regehr
guid: http://blog.regehr.org/?p=880
---

# C and C++ Aren&#8217;t Future Proof

A C or C++ program is expected to follow a collection of rules such as “don’t access out-of-bounds array elements.” There are a lot of these rules and they are listed (or are implicit) in the various language standards. A program that plays by all of these rules—called a *conforming* program—is one where we might, potentially, be able to make a guarantee about the program’s behavior, such as “This program doesn’t crash, regardless of input.” In contrast, when a C or C++ program breaks a rule, the standard doesn’t permit us to say anything about its behavior.

So where’s the problem? It comes in three parts:

1. Some of the rules imposed by the C and C++ standards are routinely violated. For example, it is not uncommon to see creation (but not dereference) of invalid pointers, signed integer overflow, and violations of strict aliasing rules.
2. Programmers expect a C/C++ implementation to behave in a certain way even when some of the rules are violated. For example, many people expect that creating an invalid pointer (but, again, not dereferencing it) is harmless. Program analyzers that warn about these problems are likely to lose users.
3. C/C++ compilers have a standard-given right to exploit undefined behaviors in order to generate better code. They keep getting better and better at this. Thus, every year, some programs that used to work correctly become broken when compiled with the latest version of GCC or Clang or whatever.

This propensity for today’s working programs to be broken tomorrow is what I mean when I say these languages are not future proof. In principle a big C/C++ program that has been extensively tested would be future-proof if we never upgraded the compiler, but this is often not a viable option.

There is a long, sad history of programmers becoming seriously annoyed at the GCC developers over the last 10 years due to GCC’s increasingly sophisticated code generation exploiting the undefinedness of signed integer overflows. Similarly, any time a compiler starts to do a better job at interprocedural optimization (this has recently been happening with LLVM, I believe) a rash of programs that does stupid stuff like not returning values from non-void functions breaks horribly. Programmers used to think it was OK to read uninitialized storage and then compilers began [destroying code that did this](http://kqueue.org/blog/2012/06/25/more-randomness-or-less/).

Let’s look at a specific example. In a recent series of posts ( [1](http://blog.frama-c.com/index.php?post/2013/01/10/Code-review-finds-minor-issue-in-Zlib), [2](http://blog.frama-c.com/index.php?post/2013/01/14/Why-verify-zlib), [3](http://blog.frama-c.com/index.php?post/2013/01/16/Bad-zlib-bad)), Pascal Cuoq has been using a formal verification tool called [Frama-C](http://frama-c.com/) to verify [zlib](http://www.zlib.net/). Why zlib? First, it’s not that big. Second, it’s ubiquitous. Third, it is believed to be high quality—if we ignore a build problem on Solaris, the last security issues were fixed in 2005. I would guess that it would be difficult to find a widely-used library that is clearly more solid than zlib.

So what kinds of problems has Pascal found in this solid piece of C code? Well, so far nothing absolutely awful, but it does appear to [create invalid pointers and to compare these against valid pointers](http://blog.frama-c.com/index.php?post/2013/01/16/Bad-zlib-bad). Is this bad? That depends on your point of view. It is possible (and indeed likely) that no current compiler exploits this undefined behavior. On the other hand, it is not straightforward to perform formal verification of zlib unless we treat it as being written in a language that is quite similar to C, but that assigns a semantics to invalid pointers. Furthermore, a new compiler could show up at any time which does something horrible (like opening an exploitable vulnerability) any time zlib computes an invalid pointer.

Of course zlib isn’t the real problem; it’s small, and probably pretty close to being correct. The real problem is that there are billions of lines of C and C++ are out there. For every thousand lines of existing code there are probably a handful of undefined behavior time bombs waiting to go off. As we move forward, one of these things has to happen:

1. We ditch the C and C++, and port our systems code to Objective Ruby or Haskell++ or whatever.
2. Developers take undefined behavior more seriously, and proactively eliminate these bugs not only in new code, but in all of the legacy code.
3. The C/C++ standards bodies and/or the compiler writers decide that correctness is more important than performance and start assigning semantics to certain classes of undefined operations.
4. The undefined behavior time bombs keep going off, causing minor to medium-grade pain for decades to come.

I’m pretty sure I know which one of these will happen.

**UPDATE from 1/24/2013:** A comment on Hacker News pointed me to this [excellent example](http://stackoverflow.com/questions/14495636/strange-multiplication-behavior-in-guile-scheme-interpreter) of C not being future-proof. This is commonplace, folks. This particular undefined behavior, signed integer overflow, can be caught by our [IOC tool](http://embed.cs.utah.edu/ioc/) which is now integrated into Clang as -fsanitize=integer, and will be in the 3.3 release.
