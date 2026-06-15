---
title: Compiler Optimizations are Awesome
url: https://blog.regehr.org/archives/1515
published: "2017-05-30T21:08:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=1515
---

# Compiler Optimizations are Awesome

This piece, which I hadn’t gotten around to writing until now since I thought it was all pretty obvious, explains why Daniel J. Bernstein’s talk, [The death of optimizing compilers](https://cr.yp.to/talks/2015.04.16/slides-djb-20150416-a4.pdf) ( [audio](http://t.co/6lvxwxWoIR)) is wrong, and in fact compiler optimizations are extremely wonderful and aren’t going anywhere.

First, the thesis of the talk is that almost all code is either hot, and therefore worth optimizing by hand, or else cold, and therefore not worth optimizing at all (even with -O). Daniel Berlin, a compiler person at Google, [has looked at the data and disagrees](https://news.ycombinator.com/item?id=9397169). We can also refute Bernstein’s argument from first principles: the kind of people who can effectively hand-optimize code are expensive and not incredibly plentiful. If an optimizing compiler can speed up code by, for example, 50%, then suddenly we need to optimize a lot less code by hand. Furthermore, hand-optimized code has higher ongoing maintenance costs than does portable source code; we’d like to avoid it when there’s a better way to meet our performance goals.

Second, size matters. Most of the computers in the world are embedded and many of these are storage-constrained. Compiler optimization reduces code size and this phenomenon is completely independent of the hot/cold issue. Without optimization we’d have to buy more expensive deeply-embedded processors that have more on-chip flash memory, and we’d also have to throw away many of those 16 GB phones that are cheap and plentiful and fairly useful today.

Third, most future software isn’t written in C and C++ but rather in higher-level languages, which more or less by definition rely on the optimizer to destroy abstraction layers, do compile-time memory management, etc.

Finally, I claim that the economics of compiler optimization are excellent. A lot of dollars are spent each year making code run faster, either by buying hardware resources or by paying programmers to write faster code. In contrast, there are probably a few thousand people actively doing compiler optimization work, and just about everyone benefits from this. If we can centralize on fewer compiler infrastructures, like GCC and LLVM and V8, then the economics get even better.

In summary, of course there’s plenty of hot code that wants to be optimized by hand, and of course there’s plenty of cold code that sees little benefit due to optimizing compilers. But neither of these facts forms an argument against optimizing compilers, which are amazingly useful and will continue to be for the indefinite future.
