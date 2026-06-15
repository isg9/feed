---
title: on psyntax — wingolog
url: https://wingolog.org/archives/2009/03/12/on-psyntax
published: "2009-03-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/03/12/on-psyntax
---

# on psyntax — wingolog

## [on psyntax](/archives/2009/03/12/on-psyntax)

12 March 2009 11:53 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)

For the last few months I've been on a [Dybvig kick](http://www.cs.indiana.edu/~dyb/pubs.html). He writes clearly, and his papers make sense.

In the last week or two I've been poking at integrating [psyntax](https://www.cs.indiana.edu/chezscheme/syntax-case/) closer to the heart of [Guile](http://www.gnu.org/software/guile/), and in doing so have grown familiar with the syntax-case algorithms.

The algorithms themselves are described as lucidly as they can be in a [chapter](http://www.cs.indiana.edu/~dyb/pubs/bc-syntax-case.pdf) of [Beautiful Code](http://oreilly.com/catalog/9780596510046/).

On the one hand, psyntax *is* beautiful. It is written in Scheme + syntax-case, yet it transforms code from Scheme + syntax-case to primitive Scheme. It's a compiler, like ones written in Scheme that compile Scheme to native code. You have to bootstrap it with a precompiled version of itself.

On the other hand, Dybvig himself damns it with faint praise, comparing it to an earlier algorithm, [KFFD](http://www.cs.ucdavis.edu/~devanbu/teaching/260/kohlbecker.pdf):

> The syntax-case expander extends the KFFD hygienic macro-expansion algorithm with support for local syntax bindings and controlled capture, among other things, and also eliminates the quadratic expansion overhead of the KFFD algorithm. The KFFD algorithm is simple and elegant, and an expander based on it could certainly be a beautiful piece of code. The syntax-case expander, on the other hand, is of necessity considerably more complex. It is not, however, any less beautiful, for there can still be beauty in complex software as long as it is well structured and does what it is designed to do.

In other words: it works, dammit, shut up and eat your porridge!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

### One response

1. [Grant Rettke](http://www.wisdomandwonder.com/) says:[13 March 2009 8:21 PM](#429faba281e778ca2ed782cb570e613f9951c36f)

   \> In the last week or two I've been poking at integrating

   \> psyntax closer to the heart of Guile

   How so?

Comments are closed.
