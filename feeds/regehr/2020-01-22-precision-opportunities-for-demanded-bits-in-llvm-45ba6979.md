---
title: Precision Opportunities for Demanded Bits in LLVM
url: https://blog.regehr.org/archives/1714
published: "2020-01-22T15:46:41Z"
feed: regehr
guid: http://blog.regehr.org/?p=1714
---

# Precision Opportunities for Demanded Bits in LLVM

\[Although this post was written to stand by itself, it builds on the [previous one](https://blog.regehr.org/archives/1709). It is authored by Jubi Taneja, Zhengyang Liu, and John Regehr.\]

When designing computer systems, it can be useful to avoid specifying behaviors too tightly. For example, we might specify that a math library function only needs to return a result that is within some distance of the true answer, in order to avoid paying the cost of a fully precise answer when that precision is not needed. Similarly, when a processor boots we might specify that some of its registers or memory are in unknown configurations, to avoid burdening the processor designers with bringing up all hardware in known states. In hardware design we often refer to [don’t-care terms in a functional specification](https://en.wikipedia.org/wiki/Don%27t-care_term). Undefined behavior in C and C++ has a similar rationale: by avoiding specification of an implementation’s behaviors in numerous inconvenient situations, more efficient implementations become possible.

A fun thing that optimizing compilers can do is to automatically infer when the full power of some operation is not needed, in which case it may be that the operation can be replaced by a cheaper one. This happens in several different ways; the method we care about today is driven by LLVM’s *demanded bits* static analysis, whose purpose is to prove that certain bits of an SSA value are irrelevant. For example, if a 32-bit value is truncated to 8 bits, and if that value has no other uses, then the 24 high bits of the original value clearly do not matter.

[Here is a demanded-bits-based optimization](https://github.com/llvm/llvm-project/blob/release/9.x/llvm/lib/Transforms/InstCombine/InstCombineSimplifyDemanded.cpp#L659-L686) that looks for a call to the [bswap](https://llvm.org/docs/LangRef.html#llvm-bswap-intrinsics) intrinsic where only a single byte of the result is demanded, and then simply shifts that one byte into the necessary position. This isn’t an important optimization for targets that have a fast bswap instruction, but it would be useful [for targets that require bswap to be open-coded](https://godbolt.org/z/jxsXEh). In this example, a full-power 64-bit bswap for 32-bit ARM (foo1) requires more than a dozen instructions but a bswap where only the low byte is demanded (foo2) requires just two instructions.

In this post we’ll present a few examples where the precision of LLVM’s demanded bits analysis compares unfavorably with a highly precise demanded bits that we created using an SMT solver. Some of these situations may be worth fixing. The solver-based algorithm is pretty straightforward. For every bit that is an input to a computation, it asks two questions:

1. If this bit is forced to zero, does the resulting computation have the same behavior as the original, for all valuations of the remaining input bits?

2. If this bit is forced to one, does the resulting computation have the same behavior as the original, for all valuations of the remaining input bits?

If both answers are “yes” then that input bit is not demanded. Otherwise it is demanded.

Side note: “demanded bits” is a slightly confusing name for this analysis. Since static analyses always approximate, we can say that they have a may direction and a must direction. If demanded bits says that a bit is demanded, this actually means “it **may** be demanded, or perhaps not: this analysis is incapable of saying for sure.” On the other hand, if it says that a bit is not-demanded, then this should be interpreted as “this bit’s value **must** not matter, and we’re confident that we could provide a formal proof of this.” The problem here is that it is more intuitive to name an analysis after its must direction, since this is the reliable information that we can directly use to drive optimizations. So, in other words, this analysis should have been called a don’t-care analysis, as is traditional. We are getting off of our soapbox now.

Ok, now on to the examples. The code will be in Souper syntax, but hopefully it is close enough to LLVM IR that nobody will have trouble seeing what’s going on. All of these program fragments are ones that we encounter while compiling SPEC CPU 2017. The LLVM that we’re comparing against is from January 20, 2020.

When a number is multiplied by a constant, some of its high bits may not affect the answer:

**```** **%0:i32 = var** **%1:i32 = mul 40:i32, %0**

**demanded-bits from Souper: for %0 : 00011111111111111111111111111111** **demanded-bits from compiler: for %0 : 11111111111111111111111111111111**

**```**

Let’s pause here for a second and assume that you don’t believe us. And why should you? This material can be counterintuitive and we make mistakes all the time, like anyone else. So [here’s a simple program you can use to check this result using exhaustive testing](https://gist.github.com/regehr/19b2ef7206922df5d455c1280dc7e8b7) instead of a solver. With a bit of effort you should be able to use it to double-check all of the results in this post.

It is also the case that some low bits of a number may not matter, when it is going to be divided by a constant:

**```** **%0:i32 = var** **%1:i32 = udiv %0, 1000:i32**

**demanded-bits from Souper: for %0 : 11111111111111111111111111111000** **demanded-bits from compiler: for %0 : 11111111111111111111111111111111**

**```**

This one is cute:

**```** **%0:i32 = var** **%1:i32 = srem %0, 2:i32**

**demanded-bits from Souper: for %0 : 10000000000000000000000000000001** **demanded-bits from compiler: for %0 : 11111111111111111111111111111111**

**```**

And so is this one:

**```** **%0:i32 = var** **%1:i32 = mul %0, %0**

**demanded-bits from Souper: for %0 : 01111111111111111111111111111111** **demanded-bits from compiler: for %0 : 11111111111111111111111111111111**

**```**

Another flavor of lost precision comes from comparison instructions:

**```** **%0:i8 = var** **%1:i1 = ult %0, 8:i8**

**demanded-bits from Souper: for %0 : 11111000** **demanded-bits from compiler: for %0 : 11111111**

**```**

This one is fun, only the sign bits matters:

**```** **%0:i8 = var** **%1:i1 = slt %0, 0:i8**

**demanded-bits from Souper: for %0 : 10000000** **demanded-bits from compiler: for %0 : 11111111**

**```**

The above represent some low-hanging fruit. We’re leaving out some UB-related imprecision since that stuff is fiddly to work with, and also a lot of imprecision that requires looking at multiple instructions together. (Unhappily, abstract transfer functions are non-compositional: even when a collection of maximally precise functions is applied to a collection of instructions, the result is often not maximally precise.) Here are two quick examples:

**```** **%0:i16 = var** **%1:i16 = add 64:i16, %0** **%2:i16 = and 448:i16, %1**

**demanded-bits from Souper: for %0 : 0000000111000000** **demanded-bits from compiler: for %0 : 0000000111111111**

**```**

And second:

**```** **%0:i32 = var** **%1:i1 = eq 0:i32, %0** **%2:i32 = select %1, 1:i32, %0** **%3:i1 = ne 0:i32, %2**

**demanded-bits from Souper: for %0 : 00000000000000000000000000000000** **demanded-bits from compiler: for %0 : 11111111111111111111111111111111**

**```**

Here we can see that %2 can never be zero, since %2 takes the value of %0 when %0 is not zero, and takes the value 1 otherwise. This analysis result is particularly satisfying because the special case of no demanded bits means that some instructions become dead. In other words, as far as this little collection of instructions is concerned, there wasn’t any need to compute %0 in the first place, because every one of the 232 values that it could take results in %3 being true.

Which of these imprecisions are worth fixing? That is harder to answer. The potential benefit is obvious: it may allow additional optimizations to be performed. The costs of increasing precision include increased compile time, the overhead of maintaining the new code, and the risk of miscompilation if someone writes an unsound transfer function.
