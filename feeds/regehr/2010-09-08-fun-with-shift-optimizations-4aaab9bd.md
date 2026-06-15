---
title: Fun With Shift Optimizations
url: https://blog.regehr.org/archives/265
published: "2010-09-08T15:57:25Z"
feed: regehr
guid: http://blog.regehr.org/?p=265
---

# Fun With Shift Optimizations

It’s fun to see what a modern compiler can optimize and what it cannot. The other day, while working on a new piece about undefined behavior, I noticed some C compilers failing to optimize simple code based on shifts. Here are the functions:

> ```
> int shift1 (int x, int y)
> {
>   if (x>=0) {
>     return (x>>y) <= x;
>   } else {
>     return 1;
>   }
> }
>
> int shift2 (unsigned x, unsigned y)
> {
>   return (x>>y) <= x;
> }
>
> int shift3 (int x, int y)
> {
>   if (x>=0) {
>     return (x<<y) >= 0;
>   } else {
>     return 1;
>   }
> }
> ```

A C99 or C++0x compiler can optimize each of these functions to return the constant 1. Why is this? shift1() and shift2() are pretty obvious: a non-negative number, shifted right by any amount, cannot increase in value, because vacated bits are filled with zeros. shift3() is trickier: when talking about left-shift, the C99 and C++0x standards state that:

> If E1 has a signed type and nonnegative value, and E1 í— 2E2 is representable in the result type, then that is the resulting value; otherwise, the behavior is undefined.

E1 is the left operand and E2 is the right operand. This is from 6.5.7 of N1124 and 5.8 of N3126. What the standards are really saying is that when left-shifting a signed integer, the compiler is permitted to assume that you don’t shift any 1 bits into the sign bit position, or past it. Thus, left-shifting a non-negative integer always results in a non-negative integer.

None of these three functions were optimized to “return 1” by any of the most recent x86 or x64 versions of Clang, GCC, or Intel CC. Rather, the generated code does the obvious testing, branching, and shifting. What’s going on? Hard to say — maybe it’s just that these situations almost never occur in real code. Let me know if you have better ideas or if you know of a compiler that does perform these optimizations. I would expect that a solid peephole superoptimizer like [the one I outlined here](https://blog.regehr.org/archives/247#superoptimizer) would discover the optimized form of these functions.

## *Update*

This is motivated by Ben Titzer’s comment below.

Fixing `y` to be 1 does not appear to make it fundamentally easier to compile these functions. None of the compilers can exploit this extra information to compile any function to “return 1.”

Fixing `x` to be 1 does make it easier to compile these functions. ICC exploits this information to optimize shift1() and shift2() so they return a constant; gcc uses it to compile shift1() to return a constant; Clang does not figure out how to do this for any of the functions.

## *Notes*

I used ICC 11.1. Clang and GCC were SVN head from 9/8/2010. All compilers were invoked something like:

> ```
> cc -std=c99 -c -O2 shift.c
>
> ```

Typical generated code (this is from gcc on x64):

> ```
> shift1:
>   movl    $1, %eax
>   testl   %edi, %edi
>   js      .L2
>   movl    %edi, %eax
>   movl    %esi, %ecx
>   sarl    %cl, %eax
>   cmpl    %edi, %eax
>   setle   %al
>   movzbl  %al, %eax
> .L2:
>   rep
>   ret
>
> shift2:
>   movl    %edi, %eax
>   movl    %esi, %ecx
>   shrl    %cl, %eax
>   cmpl    %edi, %eax
>   setbe   %al
>   movzbl  %al, %eax
>   ret
>
> shift3:
>   movl    $1, %eax
>   testl   %edi, %edi
>   js      .L7
>   movl    %edi, %eax
>   movl    %esi, %ecx
>   sall    %cl, %eax
>   notl    %eax
>   shrl    $31, %eax
> .L7:
>   rep
>   ret
> ```
