---
title: Efficient Integer Overflow Checking in LLVM
url: https://blog.regehr.org/archives/1384
published: "2016-04-07T22:09:26Z"
feed: regehr
guid: http://blog.regehr.org/?p=1384
---

# Efficient Integer Overflow Checking in LLVM

(Here’s some [optional background reading material](http://www.cs.utah.edu/~regehr/papers/tosem15.pdf).)

We want fast integer overflow checking. Why? First, if the [undefined behavior sanitizers](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) go faster then testing goes faster. Second, when overhead drops below a certain point people will become willing to use UBSan to harden production code against integer overflows. This is already being done in [parts of Android](https://twitter.com/CopperheadSec/status/717086242966351872). It isn’t the kind of thing to do lightly: some of LLVM’s sanitizers, such as ASan, will increase an application’s attack surface. Even UBSan in trapping mode — which does not increase attack surface that I know of — could easily enable remote DoS attacks. But this is beside the point.

Let’s start with this function:

```
int foo(int x, int y) {
  return x + y;
}

```

When compiled with trapping integer overflow checks (as opposed to checks that provide diagnostics and optionally continue executing) Clang 3.8 at -O2 gives:

```
foo:
        addl    %esi, %edi
        jo      .LBB0_1
        movl    %edi, %eax
        retq
.LBB0_1:
        ud2

```

This code is efficient; [here Chandler Carruth explains why](https://gist.github.com/regehr/65252e90fb6bad82d533#gistcomment-1733637). On the other hand, signed integer overflow checking slows down [SPEC CINT 2006](https://www.spec.org/cpu2006/CINT2006/) by 11.8% overall, with slowdown ranging from negligible (GCC, Perl, OMNeT++) to about 20% (Sjeng, H264Ref) to about 40% (HMMER).

Why do fast overflow checks add overhead? The increase in object code size due to overflow checking is less than 1% so there’s not going to be much trouble in the icache. Looking at HMMER, for example, we see that it spends >95% of its execution time in a function called P7Viterbi(). This function can be partially vectorized, but the version with integer overflow checks doesn’t get vectorized at all. In other words, most of the slowdown comes from integer overflow checks interfering with loop optimizations. In contrast, GCC and Perl probably don’t get much benefit from advanced loop optimizations in the first place, hence the lack of slowdown there.

Here I’ll take a second to mention that I had to hack SPEC slightly so that signed integer overflows wouldn’t derail my experiments. The changes appear to be performance-neutral. Only GCC, Perl, and H254Ref have signed overflows. [Here’s the patch](https://blog.regehr.org/extra_files/spec-patch.txt) for SPEC CPU 2006 V1.2. All performance numbers in this post were taken on an i7-5820K (6-core Haswell-E at 3.3 GHz), running Ubuntu 14.04 in 64-bit mode, with frequency scaling disabled.

Now for the fun part: making code faster by removing overflow checks that provably don’t fire. At -O0 the SPEC INT 2006 benchmarks have 67,678 integer overflow checks, whereas at -O3 there are 38,527. So that’s nice: LLVM 3.8 can already get rid of 43% of naively inserted checks.

Let’s look at some details. First, the good news: all of the overflow checks in these functions are optimized away by LLVM:

```
int foo2(int x, int y) {
  return (short)x + (short)y;
}

int foo3(int x, int y) {
  return (long)x + (long)y;
}

int foo4(int x, int y) {
  return (x >> 1) + (y >> 1);
}

int32_t foo5(int32_t x, int32_t y) {
  const int32_t mask = ~(3U << 30);
  return (x & mask) + (y & mask);
}

int32_t foo6(int32_t x, int32_t y) {
  const int32_t mask = 3U << 30;
  return (x | mask) + (y | mask);
}

int32_t foo7(int32_t x, int32_t y) {
  const int32_t mask = 1U << 31;
  return (x | mask) + (y & ~mask);
}

```

See for yourself [here](https://godbolt.org/g/5Zr0N4). The common theme across these functions is that each overflow check can be seen to be unnecessary by looking at the high-order bits of its inputs. [The code for these optimizations is here](https://github.com/llvm-mirror/llvm/blob/release_38/lib/Transforms/InstCombine/InstCombineAddSub.cpp#L893).

Now on the other hand, LLVM is [unable to see](https://godbolt.org/g/nC1xrf) that these functions don’t require overflow checks:

```
int foo8(int x) {
  return x < INT_MAX ? x + 1 : INT_MAX;
}

int foo9(int x, int y) {
  if (x < 10 && x > -10 && y < 10 && y > -10)
    return x + y;
  return 0;
}

```

Here’s another one where the check [doesn’t get eliminated](https://godbolt.org/g/ETa4Zv):

```
void foo10(char *a) {
  for (int i = 0; i < 10; i++)
    a[i] = 0;
}

```

There’s good news: Sanjoy Das has a [couple](http://reviews.llvm.org/D18684) of [patches](http://reviews.llvm.org/D18685) that, together, solve this problem. Their overall effect on SPEC is to reduce the overhead of signed integer overflow checking to 8.7%.

Finally, this function should get one overflow check [rather than three](https://godbolt.org/g/EogDMr):

```
unsigned foo11(unsigned *a, int i) {
  return a[i + 3] + a[i + 5] + a[i + 2];
}

```

This example (or something like it) is from Nadav Rotem on twitter; his [redundant overflow check removal pass for Swift](https://github.com/apple/swift/blob/master/lib/SILOptimizer/Transforms/RedundantOverflowCheckRemoval.cpp) eliminates this sort of thing at the SIL level. I’ve done a bit of work on bringing these ideas to LLVM, and will hopefully have more to write about that later on.

In summary, signed integer overflow checks in LLVM are fast but they get in the way of the optimizers, which haven’t yet been systematically taught to see through them or eliminate them. There’s plenty of low-hanging fruit, and hopefully we can get the overhead down to the point where people can turn on overflow checking in production codes without thinking too hard about the tradeoffs.

Addenda:

I haven’t forgotten about Souper! We’ve taught it to eliminate integer overflow checks. I’ll write about that later too.

See [Dan Luu’s article](http://danluu.com/integer-overflow/) on this topic.
