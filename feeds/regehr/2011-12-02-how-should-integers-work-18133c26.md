---
title: How Should Integers Work?
url: https://blog.regehr.org/archives/641
published: "2011-12-02T17:26:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=641
---

# How Should Integers Work?

![](https://blog.regehr.org/wp-content/uploads/2011/02/add_89.png)

Integer computer math is kind of a bummer because it’s a lot more complicated than it seems like it should be. I’m specifically talking about the way that fixed-width integers wreck havoc on program logic when they overflow. There are other issues, but this seems like the big one. So: How should integers work?

C/C++’s undefined overflow semantics for signed integers is a real pain and I don’t think anyone would say this is how integers should work.

Lots of languages like Java specify two’s complement semantics for integer overflow. This is reasonably efficient (though it kills optimization opportunities in loop code) but it has the problem of not matching the behavior of mathematical integers. Which integer does Java give us when we increment 2,147,483,647? It gives the integer that is — out of all possible results — the farthest from the mathematical result. Just the other day I heard a story from the high performance computing people in my department where a parallel code worked correctly except at the largest scale, when an integer would overflow. I don’t know how they debugged this but it can’t have been fun.

Saturating integer semantics where overflows stop at the maximum or minimum representable value are useful in signal processing, but they are not used as the default integer semantics in any language that I know of. Unfortunately, too few platforms offer native support for saturating semantics; otherwise there’s a significant performance penalty. Also, a design choice needs to be made about whether saturation is sticky or not. In other words, does decrementing INT\_MAX give INT\_MAX, or does it give INT\_MAX-1?

IEEE floating point supports sticky NaN values to represent nonsensical results. Adding a NaN integer value to represent overflows would seem like a reasonable compromise between efficiency and correctness. Unfortunately, the two’s complement representation has no spare bits to represent NaN, nor of course do hardware platforms support transparent propagation of integer NaNs.

Integer types can be infinitely ranged. Bignums have a reputation for poor performance, but they can be pretty fast if a smart compiler and runtime optimistically assume that operations don’t overflow, but silently trap into the bignum library when necessary. This is clearly the right choice for high-level languages, and clearly the wrong choice for systems languages like C and C++. The problem in C/C++ is that they are used to implement OS kernels, hypervisors, and bare-metal embedded systems that will become impossible to code if memory allocation can be triggered at nearly arbitrary program points. Small embedded systems don’t have have a dynamic allocator, and OS kernels have environments like interrupt handlers where allocation is dicey at best. To summarize, the most critical software systems are also the ones where we cannot afford friendly integer semantics.

Another option is explicitly ranged integers backed by formal methods tools. Here, the programmer basically has to prove that integers cannot overflow. This is unsuitable for many programming tasks because it is too difficult, but it’s probably the right answer for safety critical systems.

Finally, we can have hybrid integer semantics that combine various options. This add flexibility but makes the programming model more complex, particularly when different kinds of integers interact.

So: How should integers work?

[This post](https://blog.regehr.org/archives/392) shows some amusing bugs in simple integer codes.
