---
title: 'research!rsc: Superoptimizer'
url: https://research.swtch.com/superopt
published: "2008-01-16T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/superopt
---

# research!rsc: Superoptimizer

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Superoptimizer Russ Cox January 16, 2008 *research.swtch.com/superopt* Posted on Wednesday, January 16, 2008.

In 1987, Henry Massalin invented the “superoptimizer,” a program that finds the shortest possible instruction sequence for a given function. The superoptimizer works by sheer dumb luck: it checks whether every possible instruction sequence up to a given length produces the desired answer. The sequences that pass this test are not guaranteed to be an exact match for the function, but they usually are and can be checked analytically by a human.

As an example of what a superoptimizer can find, here is an Intel x86 instruction sequence that starts with a signed 16-bit number in ax and ends with the sign of that number (-1, 0, or 1) in dx.

> ```
> cwd
> neg ax
> adc dx, dx
>
> ```

Massalin gives this example in his 1987 paper, “ [**Superoptimizer: a look at the smallest program**](http://doi.acm.org/10.1145/36206.36194)” (subscription required, sadly).

Superoptimizers are useful for compiler writers, but perhaps more as a source of amusement than as a time-saving device. One important way that they can be useful is to find sequences that avoid conditional branches, which get more and more expensive as processor pipelines deepen. Torbjörn Granlund and Richard Kenner used a superoptimizer to eliminate branches in gcc's output; see their paper “ [Eliminating Branches using a Superoptimizer and the GNU C Compiler](http://citeseer.ist.psu.edu/granlund92eliminating.html).”

The [Wikipedia entry for Superoptimization](http://en.wikipedia.org/wiki/Superoptimization) gives links to implementations.
