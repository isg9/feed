---
title: A Common C/C++ Core Specification rev&nbsp;2
url: https://gustedt.wordpress.com/2020/05/11/a-common-c-c-core-specification-rev-2/
published: "2020-05-11T07:48:44Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3733
---

# A Common C/C++ Core Specification rev&nbsp;2

v2 of that specification just appeared on the C comittee website:

[A Common C/C++ Core Specification rev 2](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2522.pdf)

For an introduction to the originial version see “ [A common C/C++ core specification”](https://gustedt.wordpress.com/2020/03/08/a-common-c-c-core-specification/)

Most important additions in this revision are

- three-way comparison (spaceship operator)
- “initializer” construct for captures of lambdas
- a tool for textual representation of all basic types and arrays, totext
- more attributes: core::free , core::realloc, core::noleak,

  core::writethrough, core::concurrent
- constexpr
- if with variable definitions

and some more cleanups and harmonizations between C and C++.
