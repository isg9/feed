---
title: Finding Integer Undefined Behaviors Using Clang 2.9
url: https://blog.regehr.org/archives/508
published: "2011-04-08T15:02:55Z"
feed: regehr
guid: http://blog.regehr.org/?p=508
---

# Finding Integer Undefined Behaviors Using Clang 2.9

My student Peng Li modified Clang to detect integer-related undefined behaviors in C and C++ code. We’ve [released the code here](http://embed.cs.utah.edu/ubc/), to go along with the recent LLVM 2.9 release. This checker has found problems in PHP, Perl, Python, Firefox, SQLite, PostgreSQL, BIND, GMP, GCC, LLVM, and quite a few other projects I can’t think of right now.

Of course, undefined behaviors are not all created alike. The next thing that someone should do is combine this work with a tainting engine in order to find dangerous operations that depend on the results of operations with undefined behavior. For example, take this code:

> ```
> a[1<<j]++;
> ```

If it is ever the case that 0>jâ‰¥32, then a shift-related undefined behavior occurs. The result of this operation is used as an array index — clearly this is dangerous. It may even be exploitable, depending on what a particular C/C++ implementation does with out-of-bounds shifts. Other dangerous operations include memory allocations and system call arguments.

One of my favorite examples of a project getting hosed by an integer undefined behavior is [here](http://code.google.com/p/nativeclient/issues/detail?id=245). An apparently harmless refactoring of code in Google’s Native Client introduced an undefined behavior that subverted the fundamental sandboxing guarantee. This is pernicious because the (flawed) check is sitting right there in the source code, it is only in the compiler output that the check is a nop. If their regression tests used our tester, this problem would have almost certainly been caught right away.
