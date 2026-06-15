---
title: Improving the specification of arithmetic for&nbsp;atomics
url: https://gustedt.wordpress.com/2016/07/29/improving-the-specification-of-arithmetic-for-atomics/
published: "2016-07-29T12:08:02Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2302
---

# Improving the specification of arithmetic for&nbsp;atomics

In a document to the standards committee of last year I had observed a set of unfortunate inconsistencies in C11’s specification of arithmetic operations for atomics. As you perhaps know such operations can be specified with operators `a++` (in the language part) or generic functions `atomic_fetch_add(&a, 1)` (in the library), but unfortunately the two parts had not been redacted to fit well in all places.

My new document [Minimal Suggested Corrigendum for Arithmetic on Atomic Objects](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2064.htm) will hopefully make it as proposed corrigendum into the next “bugfix” version of the standard, and hopefully this new version will see the light and the end of 2017. It will perhaps not be with the exact words as they are presented, but I think that the text already comes close to what the intent of the committee had been, and to what implementors actually do, anyhow.
