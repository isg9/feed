---
title: Solutions to Integer Overflow
url: https://blog.regehr.org/archives/1401
published: "2016-09-02T21:07:43Z"
feed: regehr
guid: http://blog.regehr.org/?p=1401
---

# Solutions to Integer Overflow

Humans are typically not very good at reasoning about integers with limited range, whereas computers fundamentally work with limited-range numbers. This impedance mismatch has been the source of a lot of bugs over the last 50 years. The solution comes in multiple parts.

In most programming languages, the default integer type should be a bignum: an arbitrary-precision integer that allocates more space when needed. Efficient bignum libraries exist and most integers never end up needing more than one machine word anyway, except in domains like crypto. As far as I’m concerned, for ~95% of programming tasks integer overflow is a solved problem: it should never happen. The solution isn’t yet implemented widely enough, but happily there are plenty of languages such as Python that give bignums by default.

When performance and/or predictability is a major consideration, bignums won’t work and we’re stuck with fixed-width integers that wrap, trap, saturate, or trigger undefined behavior upon overflow. Saturation is a niche solution that we won’t discuss further. Undefined behavior is bad but at least it enables a few loop optimizations and also permits [trapping implementations](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html). Although wrapping is an extremely poor default, there are a few good things to say about it: wrapping is efficient, people have come to expect it, and it is a good match for a handful of application domains.

Swift is a modern programming language that traps instead of providing bignums, this is also a generally sensible behavior. Why not bignums? The [About Swift](https://swift.org/about/) web page says that Swift gives “the developer the control needed in a true systems programming language,” so perhaps the designers were worried about unpredictable allocations. I’d love to see a study of the performance of best-of-breed trapping and bignum implementations on important modern applications.

The Rust developers have adopted a hybrid solution where integer overflows [trap in debug builds and wrap in optimized builds](http://huonw.github.io/blog/2016/04/myths-and-legends-about-integer-overflow-in-rust/). This is pragmatic, especially since integer overflows do not compromise Rust’s memory safety guarantees. On the other hand, perhaps as [MIR](https://blog.rust-lang.org/2016/04/19/MIR.html) matures, Rust will gravitate towards checking in optimized builds.

For safety-critical programs, the solution to integer overflow is to prove that it cannot happen using some combination of manual reasoning, testing, and formal verification. [SPARK Ada](http://www.spark-2014.org/about) and the [TrustInSoft](http://trust-in-soft.com/) analyzer are suitable for proving that integer overflows won’t occur. More work is needed to make this sort of verification scalable and less expert-intensive.

Systems programming tasks, such as building operating systems, language runtimes, and web browsers, are caught in the middle. Wrapping sucks, bignums and trapping are slow or at least perceived as slow (and you do not want to trap or allocate while handling a hardware interrupt anyway), and the codes are too large for formal verification and thorough testing. One answer is to work hard on making trapping fast. For example, Swift has a [high-level optimization pass specifically for removing integer overflow checks](https://github.com/apple/swift/blob/master/lib/SILOptimizer/Transforms/RedundantOverflowCheckRemoval.cpp), and then the LLVM optimization passes do more of this, and then the LLVM backends can lower checked math operations to efficient condition code checks, and then modern Intel processors [fuse the resulting branch-on-overflow instructions away](https://blog.regehr.org/archives/1384).

In summary, bignums should be the default whenever this is feasible, and trapping on overflow should be the backup default behavior. Continued work on the compilers and processors will ensure that the overhead of trapping overflow checks is down in the noise. Java-style wrapping integers should never be the default, this is arguably even worse than C and C++’s UB-on-overflow which at least permits an implementation to trap. In domains where wrapping, trapping, and allocation are all unacceptable, we need to be able to prove that overflow does not occur.

I’ll end up with a few random observations:

- Dan Luu wrote a piece on [the overhead of overflow checking](https://danluu.com/integer-overflow/).

- Arbitrary (fixed) width bitvectors are a handy datatype and I wish more languages supported them. These can overflow but it’s not as big of a deal since we choose the number of bits.

- Explicitly ranged integers as seen in Ada are also very nice, there’s no reason that traps should only occur at the 32-bit or 64-bit boundaries.

- The formal verification community ignored integer overflow for far too long, there’s a long history of assuming that program integers behave like mathematical integers. Things are finally better though.

**UPDATE:** I didn’t want this piece to be about C and C++ but I should have clarified that it is only signed overflow in these languages that is undefined behavior; unsigned overflow is defined to be two’s complement wraparound. While it is possible to trap on unsigned overflow — UBSan has a flag that turns on these traps — this behavior does not conform to the standards. Even so, trapping unsigned wraparounds can — in some circumstances — be useful for finding software defects. The question is whether the wraparound was intentional or not.
