---
title: How Should You Write a Fast Integer Overflow Check?
url: https://blog.regehr.org/archives/1139
published: "2014-04-29T02:27:15Z"
feed: regehr
guid: http://blog.regehr.org/?p=1139
---

# How Should You Write a Fast Integer Overflow Check?

Detecting integer overflow in languages that have wraparound semantics (or, worse, undefined behavior on overflow, as in C/C++) is a pain. The issue is that programming languages do not provide access to the hardware overflow flag that is set as a side effect of most ALU instructions. With the condition codes hidden, overflow checks become not only convoluted but also error-prone. An obvious solution — writing overflow checks in assembly — is undesirable for portability reasons and also it is not as performant as we would hope: assembly has opaque semantics to the higher-level language and the overflow checks cannot be removed in cases where the overflow can be shown to not happen.

In this post we’ll look at various implementations for this addition function:

```
/*
 * perform two's complement addition on the first two arguments;
 * put the result in the location pointed to by the third argument;
 * return 1 if overflow occurred, 0 otherwise
 */
int checked_add(int_t, int_t, int_t *);

```

This operation closely mimics a hardware add instruction. Here’s an optimal (at least in the number of instructions) implementation of it for the 64-bit case for x86-64 using the GCC/Linux calling convention:

**```** **checked_add:** **xor %eax, %eax** **addq %rsi, %rdi** **movq %rdi, (%rdx)** **seto %al** **ret**

**```**

The problem is that compilers aren’t very good at taking an implementation of checked\_add() in C/C++ and turning it into this code.

This is perhaps the simplest and most intuitive way to check for overflow:

```
int checked_add_1(int_t a, int_t b, int_t *rp) {
  wideint_t lr = (wideint_t)a + (wideint_t)b;
  *rp = lr;
  return lr > MAX || lr < MIN;
}

```

The only problem here is that we require a wider integer type than the one being added. When targeting x86-64, both GCC and Clang support 128-bit integers:

```
typedef int64_t int_t;
typedef __int128 wideint_t;
#define MAX INT64_MAX
#define MIN INT64_MIN

```

On the other hand, neither compiler supports 128-bit integers when generating code for a 32-bit target.

If we can’t or don’t want to use the wider type, we can let a narrow addition wrap around and then compare the signs of the operands and result:

```
int checked_add_2(int_t a, int_t b, int_t *rp) {
  int_t r = (uint_t)a + (uint_t)b;
  *rp = r;
  return
    (a < 0 && b < 0 && r >= 0) ||
    (a >= 0 && b >= 0 && r < 0);
}

```

This code is the equivalent bitwise version:

```
#define BITS (8*sizeof(int_t))

int checked_add_3(int_t a, int_t b, int_t *rp) {
  uint_t ur = (uint_t)a + (uint_t)b;
  uint_t sr = ur >> (BITS-1);
  uint_t sa = (uint_t)a >> (BITS-1);
  uint_t sb = (uint_t)b >> (BITS-1);
  *rp = ur;
  return
    (sa && sb && !sr) ||
    (!sa && !sb && sr);
}

```

What if we don’t have a wider type and also we don’t want to use unsigned math? Then we call this function:

```
int checked_add_4(int_t a, int_t b, int_t *rp) {
  if (b > 0 && a > MAX - b) {
    *rp =  (a + MIN) + (b + MIN);
    return 1;
  }
  if (b < 0 && a < MIN - b) {
    *rp =  (a - MIN) + (b - MIN);
    return 1;
  }
  *rp = a + b;
  return 0;
}

```

It looks odd but I believe it to be correct. You can find all of these functions (plus a crappy test harness) [in this file](https://blog.regehr.org/extra_files/checked_add.c) and x86-64 assembly code for all widths [in this file](https://blog.regehr.org/extra_files/asm_checked_add.S). Let me know if you have a checked add idiom that’s different from the ones here.

As a crude measure of code quality, let’s look at the number of instructions that each compiler emits, taking the smallest number of instructions across -O0, -O1, -O2, -Os, and -O3. Here I’m targeting x86-64 on Linux and using the latest releases of the compilers: Clang 3.4 and GCC 4.9.0. The results from compilers built from SVN the other day are not noticeably different.

8 bits16 bits32 bits64 bitsGCC checked\_add\_1()991118GCC checked\_add\_2()14212121GCC checked\_add\_3()18181818GCC checked\_add\_4()12121623Clang checked\_add\_1()55514Clang checked\_add\_2()16161414Clang checked\_add\_3()20231414Clang checked\_add\_4()18181822optimal5555

The only real success story here is Clang and checked\_add\_1() for 8, 16, and 32-bit adds. The optimization responsible for this win can be found in a function called ProcessUGT\_ADDCST\_ADD() at line 1901 of [this file from the instruction combiner pass](http://llvm.org/svn/llvm-project/llvm/tags/RELEASE_34/final/lib/Transforms/InstCombine/InstCombineCompares.cpp). Its job — as you might guess — is to pattern-match on code equivalent to (but not identical to — some other passes have already transformed the code somewhat) checked\_add\_1(), replacing these operations with an [intrinsic function that does a narrower add and separately returns the overflow bit](http://llvm.org/docs/LangRef.html#llvm-sadd-with-overflow-intrinsics). This intrinsic maps straightforwardly onto a machine add and an access to the hardware overflow flag, permitting optimal code generation.

Across both compilers, checked\_add\_1() gives the best results. Since it is also perhaps the most difficult check to mess up, that’s the one I recommend using, and also it has fairly straightforward equivalents for subtract, multiply, and divide. The only issue is that a double-width operation cannot be used to perform safe addition of 64-bit quantities on a 32-bit platform, since neither Clang nor GCC supports 128-bit math on that platform.

Ideally, authors of safe math libraries would spend some time creating code that can be optimized effectively by both GCC and LLVM, and then we could all just use that library. So far there’s one such example: Xi Wang’s [libo](https://github.com/xiw/libo), which avoids leaning so hard on the optimizer by directly invoking the llvm.x.with.overflow intrinsics. Unfortunately there’s no equivalent mechanism in GCC (that I know of, at least) for guaranteeing fast overflow checks.
