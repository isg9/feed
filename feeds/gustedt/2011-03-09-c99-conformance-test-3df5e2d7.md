---
title: C99 conformance test
url: https://gustedt.wordpress.com/2011/03/09/c99-conformance-test/
published: "2011-03-09T14:56:13Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=987
---

# C99 conformance test

I have recently tested some compiler that are publicly available for their C99 conformance. Doing that I finally came up with a relatively [simple file that tests the compiler side of the conformance](http://p99.gforge.inria.fr/p99-html/test-p99-conformance.c-example.html), not the library. Most of what looked relatively good, with the notorious exception of `inline` support that I already have described in a different post: `pcc` and modern `gcc` do well, ancient `gcc`, `tcc`, `icc` and `opencc` pump symbols for `inline` functions in every translation unit, and `clang` has no standardized way of generating such a symbol.

I have set up a [directory that reflects the status of these different tests](http://p99.gforge.inria.fr/c99-conformance). Please, anyone that is interested have a look if your favorite compiler is there, and if not, drop me a line to tell me about it.

Passing these tests is one thing, being able to compile P99 is unfortunately another. There are two of that set for which I didn’t manage to do so.

The first, TinyCC or tcc, doesn’t even claim to be C99 conforming. Still it has some extensions that should make part of C99 work, but up to now I have not been able to separate out a simple example where it fails. Well here is an example that looks sufficiently simple

```
P99_TOK_EQ(1, 1)

```

but if you look into the expansion that P99 does for this, well it ain’t as simple anymore.

The second is pcc, one of the oldest C compilers that is still around and which recently has been pushed towards C99 conformance. This simply crashes under P99.
