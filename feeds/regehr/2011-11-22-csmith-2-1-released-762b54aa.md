---
title: Csmith 2.1 Released
url: https://blog.regehr.org/archives/631
published: "2011-11-22T19:37:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=631
---

# Csmith 2.1 Released

![](https://blog.regehr.org/wp-content/uploads/2011/11/csmith.png) We’ve released version 2.1 of [Csmith](http://embed.cs.utah.edu/csmith/), our random C program generator that is useful for finding bugs in compilers and other tools that process C code. The total number of compiler bugs found and reported due to Csmith is now more than 400. All Csmith users should strongly consider upgrading.

New features in this release — all implemented by Xuejun Yang — include:

- By default, functions and global variables are marked as “static,” permitting compilers to optimize more aggressively in some cases.
- We now try harder to get auto-vectorizers and other loop optimizers in trouble by generating code that is more idiomatic and therefore more likely to be optimized. In particular, array indices are in-bounds by construction instead of by using % operators.
- Unions are supported. Generating interesting but conforming uses of unions was not as easy as we’d have hoped.
- The comma operator is supported, as in x = (y, 1, z, 3).
- Embedded assignments are supported, as in x = 1 + (y = z).
- The pre/post increment/decrement operators are supported.
- A `--no-safe-math` mode was added, which avoids calling the safe math wrappers. This is useful when trying to crash compilers but the resulting executables should not be run since they are very likely to have undefined behavior.

These features, other than `--no-safe-math`, are turned on by default. They have found quite a few new compiler bugs; the most interesting is perhaps [this one](https://blog.regehr.org/archives/558).

An excellent recent development is that Pascal Cuoq, one of the main [Frama-C](http://frama-c.com/) developers, has done a lot of cross-testing of Frama-C and Csmith. I’m not sure what the final score was, in terms of bugs found in Csmith vs. bugs found in Frama-C, but the virtuous cycle has increased the quality of both tools. Frama-C is totally great and I recommend it to people who are serious about writing bulletproof C code.
