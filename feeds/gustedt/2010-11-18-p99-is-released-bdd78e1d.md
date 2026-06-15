---
title: P99 is released
url: https://gustedt.wordpress.com/2010/11/18/p99-is-released/
published: "2010-11-18T08:47:46Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=605
---

# P99 is released

## [P99](http://p99.gforge.inria.fr/) – Preprocessor macros and functions for C99

P99 is a toolbox of macro and function definitions that ease the programming in modern C, aka C99. By using new tools from C99 it implements default arguments for functions, scope bound resource management, transparent allocation and initialization, …

The complexity of the tools ranges from very simple (but convenient) macros such as [P99\_INIT](http://p99.gforge.inria.fr/p99-html/group__integers_ga9027c7b4da28cf06bb2d369a6a359b28.html#ga9027c7b4da28cf06bb2d369a6a359b28) to relatively complex ones such as [P99\_UNWIND\_PROTECT](http://p99.gforge.inria.fr/p99-html/group__preprocessor__blocks_ga3a751607e9ac837ee9c95efbaac5721f.html#ga3a751607e9ac837ee9c95efbaac5721f).

*P99 is not a library* but just a set of include files. You may include the whole by just using *“p99.h”* or cherry pick individual parts to your needs. You will not have to link against a special library the “only” prerequisite is that your compiler supports modern C, aka C99, to a wide extent.

So far I have tested P99

- on linux systems
- with INTEL 32 / 64 bit and ARM processors
- with four different compilers: `gcc`, `clang`, `opencc` and `icc`
- with code from an internal project.

If you are developing for another setting I would be very much curious to hear of your experience with P99.

P99 can be downloaded at [p99.gforge.inria.fr](http://p99.gforge.inria.fr/). It is licensed under the [QPL](https://en.wikipedia.org/wiki/Q_Public_License).
