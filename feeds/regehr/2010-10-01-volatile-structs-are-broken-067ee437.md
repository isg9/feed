---
title: Volatile Structs Are Broken
url: https://blog.regehr.org/archives/274
published: "2010-10-01T03:46:25Z"
feed: regehr
guid: http://blog.regehr.org/?p=274
---

# Volatile Structs Are Broken

For at least a year we’ve taken a break from reporting volatile bugs to C compiler developers. There were various reasons: the good compilers were becoming pretty much correct, we faced developer apathy about volatile incorrectness, our volatile testing tool had some weird problems, and basically I just got bored with it. Today my PhD student [Yang Chen](http://www.cs.utah.edu/~chenyang/) got a new and improved volatile testing tool working, with the goal of testing the volatile correctness of the new [CompCert](http://compcert.inria.fr/) 1.8 for x86. Instead, he immediately found some distressing bugs in other compilers. As a side note [Pin](http://www.pintool.org/), the basis for Yang’s new tool, is pretty damn cool.

Given this C code:

> ```
> struct S1 {
>   int f0;
> };
>
> volatile struct S1 g_18;
>
> void func_1 (void) {
>   g_18 = g_18;
> }
>
> ```

All recent versions of GCC for x86 (I tried 3.0.0, 3.1.0, 3.2.0, 3.3.0, 3.4.0, 4.0.0, 4.1.0, 4.2.0, 4.3.0, 4.4.0, 4.5.0, and today’s SVN head) and Clang for x86 (I tried 2.6, 2.7, and today’s SVN head) give equivalent wrong code:

> ```
> regehr@john-home:~$ gcc-4.4 -fomit-frame-pointer -O -S -o - small.c
> func_1:
>     rep
>     ret
> ```

The generated code must both read from and write to the struct. Aren’t there any embedded systems broken by this?  LLVM-GCC 2.1 through 2.7 and Intel CC generate correct code.

And here is the obligatory link to our 2008 paper [Volatiles Are Miscompiled, and What to Do about It](http://www.cs.utah.edu/%7Eregehr/papers/emsoft08-preprint.pdf). But obviously we’re not doing enough. Bugzilla links are [here](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=45852) and [here](http://llvm.org/bugs/show_bug.cgi?id=8267). (Update: fixed in LLVM within three hours of being reported.  Cool!)
