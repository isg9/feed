---
title: Bit Puzzle Results
url: https://blog.regehr.org/archives/655
published: "2011-12-21T21:44:37Z"
feed: regehr
guid: http://blog.regehr.org/?p=655
---

# Bit Puzzle Results

This is the followup piece to Monday’s [bit puzzle](https://blog.regehr.org/archives/651). First off, the spec for this problem wasn’t clear what to return when both arguments are the same. But let’s look at Carlos’s comment about where this code would be used:

> If you think of a binary number as a sequence of binary tree decisions on the full range of possible numbers, this bit operation gives you the most specific common ancestor on this tree.

This use case makes it seem pretty clear that just returning the argument is the right result. So that’s what I’ve assumed here.

These are the basic approaches to the problem:

1. Construct the output by iterating over bits.
2. Compute the result using “standard bit tricks” built on operations found on all common processors.
3. If the target processor has fast “find first set bit” instructions (e.g. modern x86/x64 has bsf/bsr), these can be applied to the XOR of the arguments, and after that the output can be generated straightforwardly.

Code for approach 1 can be found in my original post. Solutions using approach 2 are exemplified by code from Eddie Kohler, Andrew Hunter, and others:

```
uint64_t high_common_bits_portable(uint64_t a, uint64_t b)
{
  uint64_t x = a ^ b;
  x |= x >> 1;
  x |= x >> 2;
  x |= x >> 4;
  x |= x >> 8;
  x |= x >> 16;
  x |= x >> 32;
  return (a & ~x) | (x & ~(x >> 1));
}

uint64_t low_common_bits_portable(uint64_t a, uint64_t b)
{
  uint64_t x = (a ^ b) & -(a ^ b);
  return (a & (x - 1)) | x;
}

```

The first one is really nice.

Here is code implementing approach 3:

```
uint64_t high_common_bits_intrinsic(uint64_t a, uint64_t b)
{
  if (a == b) return a;
  uint64_t bit = 1ULL << 63 - __builtin_clzll(a ^ b);
  return (a & ~(bit - 1)) | bit;
}

uint64_t low_common_bits_intrinsic(uint64_t a, uint64_t b)
{
  if (a == b) return a;
  uint64_t bit = 1ULL << __builtin_ctzll(a ^ b);
  return (a & bit - 1) | bit;
}

```

These are mostly the code left by Benjamin Kramer, but modified to avoid invoking the builtin function with a zero argument, which the GCC documentation says has undefined behavior. This is the same approach that I used when solving this problem, though Benjamin did a better job with the masking code. This code doesn’t use the actual bsf/bsr instructions but rather GCC intrinsics that are supposed to compile to whatever efficient “find first set bit” primitive exists on the target platform.

Now that we have three implementations of each function, let’s work on testing them. We first need test vectors. It seems clear that making the inputs independently random 64-bit integers is wrong. My approach was to use a random 64-bit number for one of the inputs, and to construct the other input by flipping 0..63 random bits of the first argument. It’s possible that this isn’t great, I don’t know — but it’s not going to matter that much when benchmarking approaches 2 and 3 since they contain almost no data-dependent control.

My approach was to generate 1000 inputs, start a timer, run a function over all of the inputs, then stop the timer. 1000 values seemed reasonable since these arrays fit into the L1 of most any modern machine. Each function runs over the array five times and the fastest result is reported. [The code is here](http://www.cs.utah.edu/~regehr/bitdiff.c).

Here are the results from a Core i7 920 running Linux in 64-bit mode:

**\[WRONG RESULTS REMOVED — SEE BELOW\]**

My i7 920 is based on the Nehalem architecture — one off from Intel’s latest. If anyone has a Sandy Bridge processor (Core i7 2600 or 2700, most likely) sitting around, please run the code and send me results, but please be careful to [turn off processor power management and take other appropriate timing precautions](https://blog.regehr.org/archives/330). I’d also like to see results for Clang, GCC, and ARM’s compiler on a modern ARM processor.

Here are some statically linked executables for x86-64 Linux: [GCC](http://www.cs.utah.edu/~regehr/bitdiff-gcc), [Clang](http://www.cs.utah.edu/~regehr/bitdiff-clang), [Intel CC](http://www.cs.utah.edu/~regehr/bitdiff-icc).

Finally, thanks, everyone, for the great responses!

**UPDATE:**

Argh, my first attempt failed to stop the compiler from inlining the bit functions into the test harness. (Thanks for pointing this out, Richard and Chris!) The results were not technically incorrect, but they didn’t correspond to a reasonable use case for these functions. In particular, the Intel compiler was able to use SIMD to massively optimize operations on sequential array elements. After turning off inlining, the results are:

```
[regehr@gamow code]$ clang -O3 -march=corei7 -mtune=corei7 bitdiff.c -o bitdiff ; ./bitdiff

high slow      : time =  52.7 (ignore this: 10649328621511951360)
high portable  : time =  16.6 (ignore this: 10649328621511951360)
high intrinsic : time =   9.5 (ignore this: 10649328621511951360)
low slow       : time =  53.4 (ignore this: 9949931644094715106)
low portable   : time =   7.0 (ignore this: 9949931644094715106)
low intrinsic  : time =   8.6 (ignore this: 9949931644094715106)

[regehr@gamow code]$ current-gcc -Ofast -march=corei7 -mtune=corei7 bitdiff.c -o bitdiff ; ./bitdiff

high slow      : time =  48.4 (ignore this: 8093869745297464062)
high portable  : time =  15.4 (ignore this: 8093869745297464062)
high intrinsic : time =   9.5 (ignore this: 8093869745297464062)
low slow       : time =  48.0 (ignore this: 8862089369885824930)
low portable   : time =   8.0 (ignore this: 8862089369885824930)
low intrinsic  : time =   8.5 (ignore this: 8862089369885824930)

[regehr@gamow code]$ icc -fast bitdiff.c -o bitdiff ; ./bitdiff

high slow      : time =  51.1 (ignore this: 15670527381357769456)
high portable  : time =  18.0 (ignore this: 15670527381357769456)
high intrinsic : time =  12.3 (ignore this: 15670527381357769456)
low slow       : time =  48.3 (ignore this: 17922617506918019178)
low portable   : time =   9.0 (ignore this: 17922617506918019178)
low intrinsic  : time =   9.9 (ignore this: 17922617506918019178)
```

Times are in CPU cycles per call. The answer appears to be: regardless of compiler, the intrinsic version for high\_common\_bits gives about a 50% speedup. For low\_common\_bits, it doesn’t matter which version you use.

Finally, here’s what Clang gives for the version of low\_common\_bits using intrinsics:

```
low_common_bits_intrinsic:
        movq    %rdi, %rax
        cmpq    %rsi, %rax
        je      .LBB5_2
        xorq    %rax, %rsi
        bsfq    %rsi, %rcx
        movl    $1, %edx
        shlq    %cl, %rdx
        leaq    -1(%rdx), %rcx
        andq    %rax, %rcx
        orq     %rdx, %rcx
        movq    %rcx, %rax
  .LBB5_2:
        ret
```

I think the final movq could be avoided, but otherwise this looks good.
