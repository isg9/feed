---
title: Proposal for Automated Compiler Bug Reports
url: https://blog.regehr.org/archives/563
published: "2011-08-02T20:03:06Z"
feed: regehr
guid: http://blog.regehr.org/?p=563
---

# Proposal for Automated Compiler Bug Reports

*\[Yesterday I submitted a proposal to Google for a modest amount of money to work on turning large random programs that expose compiler flaws into concise bug reports. Below is a transcription that is mostly faithful (citations are omitted and I changed the example bug report from a floating figure into inline text). Feedback is appreciated.\]*

# Abstract

Csmith, our random testing tool for C compilers, has helped us discover and report nearly 400 previously unknown compiler bugs, mostly in GCC and LLVM. More than 90% of these bugs have now been fixed by compiler developers. Moreover, most of these bugs were in the “middle ends” of the compilers, meaning that they impact multiple programming languages (Go, C++, Ada, etc.) as well as many target architectures.

By proactively finding compiler bugs missed by existing test suites, we help compiler teams eliminate flaws that would otherwise make it into deployed tools. Though it is uncommon to encounter a serious compiler bug, when one does bite a development effort a lot of time is typically wasted. We help by performing nearly continuous testing of the GCC and LLVM development versions. After reporting hundreds of bugs, we appear to have reached a steady state where both compilers are substantially correct but every week or two a new bug is found.

The process of turning a bug-triggering compiler input into a good bug report is tedious and time consuming, while also requiring technical skill and good taste. We propose removing humans from this part of the picture by automating the construction of compiler bug reports.

# Research Goals

The goal of the proposed work is to enable a workflow where someone installs our software on a fast machine and tells it how to build the latest compiler version and where to send bug reports.

The machine is now totally unattended. Each day it will build the latest compiler version and spend the rest of the day testing it. When a new bug is discovered, it will be reported. The bug reports will look like this:

> Bug-triggering program:
>
> ```
> extern int printf (const char *, ...);
>
> static int *g_135 = &g_16[0];
> static int *l_15 = &g_16[0];
>
> int main (void) {
>   g_16[0] = 1;
>   *g_135 = 0;
>   *g_135 = *l_15;
>   printf("%d\n", g_16[0]);
> }
> ```
> - Expected output — produced by GCC 3.4, GCC 4.1, Intel CC 11.0, Clang 2.8, and Microsoft VC9 — is “0”.
> - Using r156486 for i686-pc-linux-gnu, “gcc -O” produces “1”. The same version at -O0 produces the expected output.
> - First version of GCC containing this bug is r147613.
> - Probable fault location: the dead store elimination pass.

The manually generated report for this bug is [here](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=42952).

The problem we need to solve is taking a large chunk of random C code (empirically, the sweet spot for randomized compiler bug finding is around 80KB of code) and turning it into an actionable bug report. Our three years’ of experience reporting compiler bugs has given us a good working understanding of what compiler developers want and need in a bug report. The rest of this proposal explains how the different parts of this report will be produced.

# The Most Important Problem: Test Case Minimization

The most important problem is test case minimization: turning a large pile of random C code into a small failure-triggering input like the one above. The *ddmin* Delta debugging algorithm is the best-known solution for test case minimization. In a nutshell, a Delta debugger is just a greedy search that works by iteratively removing a chunk of the input and checking if the new input still triggers the bug. Delta can be significantly optimized by progressively removing smaller and smaller chunks of the input and also by coordinating its choice of chunks with the structure of the test input. The best-known Delta implementation, by Wilkerson, incorporates both of these optimizations.

It works but has two serious problems when used to reduce the size of C programs. First, it gets stuck at local minima that are far from the (apparent) true minimum sizes for failure-inducing compiler inputs due to being line-based. Many interesting reduction operations for C programs need to be aware of its token-level and AST-level structure. Second, Wilkerson’s Delta usually creates reduced test cases that invoke undefined behavior. The GCC developers — to put it mildly — do not appreciate bug reports containing test cases that rely on undefined behavior.

We propose investigating two different approaches to fixing these problems. The first approach has two parts:

- A C-aware Delta debugger that will significantly improve on the Wilkerson Delta by first performing local transformations such as reducing x+y to x or x to 0, and second by performing global simplifications such as removing an argument not only from a function definition, but also from all uses of the function, removing a level of indirection from a pointer, and performing inline substitution of simple function bodies.
- We are working with Chucky Ellison at UIUC who is developing an executable semantics for C that robustly detects most kinds of undefined behavior (including several, such as unsequenced updates to an lvalue, that are largely undetected by other tools). His tool is a research prototype; we are helping refine it into a useful part of a Delta debugging loop.

Our second new approach to testcase reduction takes an entirely different approach: using Csmith itself to perform test case minimization. Our insight is that this will work because Csmith understands not only the structure of C, but also how to avoid introducing undefined behavior. Drawbacks of the Delta-in-Csmith approach are that it increases the complexity of an already complex tool and that it cannot be used to reduce failure-inducing C programs that did not come from Csmith in the first place. We believe that it is worth exploring both approaches.

# Secondary Research Problems

Although our major focus will be on creating reportable test cases, we also plan to attack the following problems. First, making a robust guess at the first revision that contains a compiler bug. We have already tried using a naive binary search to find the first broken version and too often, the result of the search is a version that contains an incidental change that merely exposes the bug. By randomly exploring a variety of similar test cases and compiler options, we hope to robustly locate the first version that contains the bug.

Second, we will perform fault localization—try to guess where in the compiler the problem lies. When the problem lies in an optional part of the compiler (i.e., an optimization pass) this is fairly easy using Whalley’s work. When the bug lies in the frontend or backend, the situation is more difficult and is beyond the scope of this proposal.

# Outcome

Evaluating this work is easy. The only important question is: Are our bug reports good enough that they motivate compiler developers to fix the bugs? We will answer this question by continuing to report bugs to major open source compiler projects like GCC and LLVM. Furthermore, our test case reduction and related tools will be released as open source as part of the already open-sourced [Csmith](http://embed.cs.utah.edu/csmith).

# Relation to Previous Work

We are already taking advantage of as much previous work as seems to be available. Mainly, this consists of Zeller’s Delta debugging papers and a handful of follow-up papers that have appeared over the last ten years, in addition to Wilkerson’s Delta implementation and Ellison’s executable C semantics. No previous compiler bug-finding effort (that we are aware of) has resulted in anything close to the nearly 400 bugs we have reported, and our guess is that for this reason nobody has yet had to develop good tool support for automated bug reports.
