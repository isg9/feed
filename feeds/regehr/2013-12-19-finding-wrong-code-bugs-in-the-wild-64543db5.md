---
title: Finding Wrong-Code Bugs in the Wild
url: https://blog.regehr.org/archives/1067
published: "2013-12-19T06:43:19Z"
feed: regehr
guid: http://blog.regehr.org/?p=1067
---

# Finding Wrong-Code Bugs in the Wild

Compiler developers already have some decent ways to find bugs:

- a self-hosting compiler won’t bootstrap if it’s too buggy
- any serious compiler has a test suite
- commercial [compiler validation suites](http://www.plumhall.com/stec.html) are available
- random testcase generators like [Csmith](http://embed.cs.utah.edu/csmith/) and [CCG](https://github.com/Merkil/ccg) can expose subtle bugs

Of course, these tests don’t find all of the bugs. The remaining bugs will make it into a released compiler version at which point they either:

- remain latent, because they are not triggered by the compiler’s users
- make life difficult, because they are triggered by the compiler’s users

This piece is about finding the last category of compiler bugs without doing it the hard way. The hard way to find a wrong-code bug is to notice an application misbehaving—probably late in development, and perhaps after deployment—and then through much hard work, to narrow down the source of the problem. This is how we do things now and it’s no fun, especially in domains like embedded systems where debugging support might be rudimentary and where the compiler might be much buggier than, for example, a modern GCC for x86-64.

# Doing the Job Right

The gold standard for finding wrong-code bugs in the wild is *translation validation* where automated formal methods are used to ensure that the compiler’s output implements the computation described by its input. Translation validation is not at all easy and there have not been very many examples where it has been convincingly applied to real code emitted by an optimizing compiler. [This recent paper](http://www.nicta.com.au/pub?doc=6449) about validating the translation of the seL4 kernel is very nice work, though it deals with a fairly small code base that is implemented in a subset of C. Their basic strategy is to decompile ARM binaries into logic, to translate C code into logic, and then to use the Isabelle/HOL theorem prover, assisted by an SMT solver, to show that the ARM code refines the C code.

Translation validation is a fairly pragmatic way to get good guarantees about compiler correctness because it does not require us to throw out our existing compiler in favor of an artifact like CompCert which—although it is one of the best things to come out of the compiler research community in the last 20 years—probably does not allow compiler developers to move fast enough to deal with all of the new optimizations, new language features, and new hardware targets that we keep throwing at them. In contrast, if a translation validator runs into a hardware feature or code construct that it does not understand, it will simply fail to validate that function; no big deal.

A drawback of translation validation is that it requires a mechanized semantics for the source language. Although some formal semantics for C have become available in the last few years, the real world is messier: systems are implemented in idiosyncratic programming languages that sort of mostly overlap with C or C++. For example, the Linux source code (for a given platform) only makes sense if we understand the dialect implemented by GCC and also a number of details about the target platform. Making things worse, GCC implements many, many dialects: OpenMP, C with pthreads, C with vector instrinsics, C with inline ARM assembly, C without the strict aliasing rules, etc. It is not going to be very easy or fun to formalize the subset of these dialects that people actually care about. C++11, as far as I know, remains largely unsullied by formal semantics, and I don’t think all of any earlier C++ has been formalized either.

A final drawback of translation validation is that it becomes harder as optimizations become more clever. Even routine transformations like loop unrolling and software pipelining can be tricky, and little work has been done on automated translation validation for aggressive autovectorizers and interprocedural optimizations (much less link-time optimizations). A workaround that has been used is to have the compiler tell the translation validator what transformations it performed and in what order; this is certainly reasonable for a research project and may also be workable in practice.

**Summary:** For now, translation validation remains more a research topic than a practical technology. It’s not too hard to imagine a future where certain language dialects enjoy support from a translation validation tool.

# Doing Without a Semantics for the Source Language

Most compiler bugs do not impact unoptimized compilations. Also, few programs are miscompiled by multiple compilers. Given these facts, we might hope to forget about the source language and rather look for inequivalence between different compiled versions of the same source code. For example, we might check that “clang -O0” and “gcc -O2” emit equivalent code. If so, the vast majority of compiler bugs would be ruled out. By analogy with differential testing, we’ll call this *differential translation validation*.

Equivalence checking for binaries is a hard and important problem that has been the subject of an oddly small amount of research. Here’s [a recent paper](http://www.stanford.edu/~sharmar/pubs/ddec.pdf) that looks promising, though the method was not evaluated on large inputs. My guess is that as technologies such as SAT/SMT solvers and mechanized semantics for machine codes continue to improve, equivalence checking for binaries will get considerably easier.

Optimizations that make regular translation validation difficult will also cause problems for differential translation validation. Then there is the additional problem of coping with languages that permit multiple inequivalent translations of the same source program. For example, consider this C code:

```
int foo (int x)
{
  return (x+1) > x;
}

```

Here is one valid translation to x64:

> **```** **_foo:** **cmpl $2147483647, %edi** **setne %al** **movzbl %al, %eax** **ret**
>
> **```**

And here is another:

> **```** **_foo:** **movl $1, %eax** **ret**
>
> **```**

These return different results when the argument is `INT_MAX`. The difference is that the second compiler has taken advantage of the undefinedness of signed integer overflow whereas the first one has not. There are various ways to deal with this problem; one of them would be to check for equivalence after conditioning the inputs with the weakest precondition that avoids undefined behavior. I don’t know how practical that is, but automated weakest precondition tools do exist. Here the weakest precondition is obviously `x != INT_MAX`. Analogous problems can occur due to unspecified behavior. For example:

```
void foo (int x, int y)
{
}

int a;

int bar (void)
{
  return a=1;
}

int baz (void)
{
  return a=2;
}

int unspecified (void)
{
  foo (bar(),baz());
  return a;
}

```

The current version of GCC for x86-64 has `unspecified()` return 1 whereas the current Clang has it return 2. Both are correct. If differential translation validation fails for this reason, we have almost certainly found a bug in the application code. If we haven’t found an application bug, that means we don’t care whether the function returns 1 or 2, in which case we’re probably dealing with a code base where translation validation is overkill.

Some codes contain large functions and other codes are compiled into large functions after inlining has been performed. It seems likely that almost any conceivable toolchain for differential translation validation will sometimes be thwarted by huge functions. Here’s an idea that I have for coping with this situation:

1. Compile the program using the production compiler and also the reference compiler

2. Perform differential translation validation; if all functions validate, we’re finished

3. For every function that failed to validate, either disable some optimizations or else split the function into two parts that can be compiled separately

4. go back to step 1

The idea is that the compiler user doesn’t care whether validation fails due to a compiler bug or a limitation of the validation tool; either way, the issue gets solved automatically. Thus, the advantage of this approach is that it automatically adapts to cope with weaknesses in the tools. A drawback is that turning off optimizations and splitting functions will result in slower code, forcing us to choose between fast code and validated code. Another problem is that this technique adds a source-to-source function splitting tool to the trusted computing base. My opinion is that—at least for C—splitting a function is simple enough that a trustable tool can be created, and perhaps someone with a C semantics can prove it correct.

**Summary:** I believe research on differential translation validation would be worthwhile. For languages like C/C++ where there are numerous dialects in common use, this technique has the advantage of not requiring a semantics for each source language dialect. Machine language semantics are relatively easy to create and, looking forward, are probably going to exist anyway for every architecture that matters.

# Tests Instead of Proofs

If differential translation validation is too difficult, we can back off and use a testing-based approach instead. In general, this means we’re no longer guaranteed to find all bugs, but keep in mind that there is some [interesting overlap between equivalence testing and equivalence verification](http://www.pgbovine.net/PhD-memoir/ucklee-cav-2011.pdf). As this paper suggests, symbolic test case generation seems like a good starting point for an equivalence checking effort. However, we’d like to apply the technique to machine instructions rather than source code or LLVM instructions. As with differential translation validation, we’re going to have to deal with inequivalences resulting from compiler exploitation of undefined behavior. One way to do this would be to check that each concrete input satisfies the weakest precondition for the function; another way would be to test each concrete input by executing it on an instrumented version of the function that dynamically checks for undefined behavior.

So far I’ve been talking about checking individual functions. A different approach—and a super easy one at that—is to test a whole application. This is easier because the preconditions of each function are (ostensibly, at least) automatically satisfied by the function’s callers. One really easy thing we can do is compile a program at multiple optimization levels and make sure its test suite always passes. Some programs support debug builds that effectively accomplish this goal, but I’m often surprised how many programs don’t have any explicit support for unoptimized compiles; this is useful for a number of reasons. First, there’s the off chance that a failure in the test suite will reveal a compiler bug. For example, the [GMP page](https://gmplib.org/) says:

> Most problems with GMP these days are due to problems not in GMP, but with the compiler used for compiling the GMP sources. This is a major concern to the GMP project, since an incorrect computation is an incorrect computation, whether caused by a GMP bug or a compiler bug. We fight this by making the GMP testsuite have great coverage, so that it should catch every possible miscompilation.

Second, there’s a somewhat larger chance that undefined behavior in the application will be revealed by testing at different optimization levels. Third, in an unoptimized build the compiler will fail to destroy undefined operations such as use of uninitialized variables that can sometimes be deleted by the optimizer; when these operations are present at runtime they can be detected by binary analysis tools such as Valgrind. If running the test suite does not seem sufficient, additional test cases can be generated using a tool like [Klee](http://klee.llvm.org/).

**Summary:** Testing for equivalence is doable using technologies that exist today, but it’s not clear how much our confidence in the compiler’s translation should increase if these kinds of testing do not reveal any problems.

# Finding Bugs that Haven’t Bit Us Yet

So far this post has been about finding bugs that actually affect our software. For application developers, these are the only compiler bugs that matter. On the other hand, as someone who tests compilers, I’m interested in finding bugs that don’t affect, for example, the Linux kernel right now, but that might affect it next month. Generative fuzzing—generating random programs from scratch using tools like Csmith and CCG—is good for improving overall compiler robustness but is not the right answer for targeted fuzzing. Rather, we should use a mutation-based fuzzer that takes Linux kernel code, changes it a bit, and sees if the new code triggers a compiler bug. As long as the mutations don’t look too different from changes that a human developer would make, this technique would seem like a good way of flushing out bugs that might matter soon. One question we might ask is how to avoid undefined behaviors when mutating source code; I’ve recently learned about some good tricks for doing this that I’ll write up in a subsequent post.
