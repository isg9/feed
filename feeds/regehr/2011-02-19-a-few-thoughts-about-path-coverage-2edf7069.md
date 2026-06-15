---
title: A Few Thoughts About Path Coverage
url: https://blog.regehr.org/archives/386
published: "2011-02-19T20:46:42Z"
feed: regehr
guid: http://blog.regehr.org/?p=386
---

# A Few Thoughts About Path Coverage

[Klee](http://klee.llvm.org/) isn’t the only tool in its class, nor was it the first, but it’s open source and well-engineered and it works. Klee’s goal is to generate a set of test inputs that collectively induce path coverage in a system under test. One of the scenarios I was curious about is the one where Klee achieves full path coverage but still fails to expose a bug, for example because a stronger kind of coverage such as value coverage is required. Coming up with these examples is easy. Consider the following function for computing the number of bits set in a 64-bit word:

> ```
> const int pop8[256] = {
>   0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4,
>   1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5,
>   1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 5, 5, 6,
>   1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 5, 5, 6,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 5, 5, 6,
>   3, 4, 4, 5, 4, 5, 5, 6, 4, 5, 5, 6, 5, 6, 6, 7,
>   1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 4, 5, 6,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 5, 5, 6,
>   3, 4, 4, 5, 4, 5, 5, 6, 4, 5, 5, 6, 5, 6, 6, 7,
>   2, 3, 3, 4, 3, 4, 4, 5, 3, 4, 4, 5, 4, 5, 5, 6,
>   3, 4, 4, 5, 4, 5, 5, 6, 4, 5, 5, 6, 5, 6, 6, 7,
>   3, 4, 4, 5, 4, 5, 5, 6, 4, 5, 5, 6, 5, 6, 6, 7,
>   4, 5, 5, 6, 5, 6, 6, 7, 5, 6, 6, 7, 6, 7, 7, 8,
> };
> ```
>
> ```
> int popcount (unsigned long long x) {
>   int c=0;
>   int i;
>   for (i=0; i<8; i++) {
>     c += pop8[x&0xff];
>     x >>= 8;
>   }
>   return c;
> }
> ```

This function admits only a single path of execution, and therefore Klee comes up with only a single test case: the 64-bit word containing all zero bits. This popcount function is, in fact, wrong (one of the values in the table is off by one) but Klee’s input does not trigger the bug. At this point we’re more or less stuck — Klee cannot find the bug and we’re back to code inspection or random testing or something. But what if we have a second popcount implementation?

> ```
> int popcount_slow (unsigned long long x) {
>   int c=0;
>   int i;
>   for (i=0; i<64; i++) {
>     c += x&1;
>     x >>= 1;
>   }
>   return c;
> }
> ```

Unlike the previous implementation, this one is correct. Like the previous implementation, it has only a single path — so again Klee will give us just one test case. The way to find an input where the two functions return different answers is to just ask Klee directly:

> ```
> assert (popcount(x) == popcount_slow(x));
> ```

Now there are two paths, a succeeding one and a failing one, and it takes Klee just a few seconds to find inputs that execute both paths. This is a pretty impressive result given the large input space. Of course, in this particular case random testing using uniformly-distributed 64-bit inputs finds this bug very quickly since approximately one in eight inputs triggers the bug. However, we could write code where the probability of a random input triggering the bug is arbitrarily small (note however that fooling a whitebox test case generator is not totally trivial: many of the tempting ways to add a bug would also add a new code path, which Klee would duly explore).

The interesting thing about path coverage is that it’s massively insufficient for finding all bugs in real code, and yet it is also impossible to achieve 100% feasible path coverage on real software systems due to the path explosion problem. The conclusion I draw is that future Klee-like tools should provide the option of targeting even stronger kinds of coverage (path + boundary value coverage, for example) and that these tools will be most useful in unit-testing of smallish codes.

The problem in testing my code above stems from its use of a lookup table. Perhaps people who test software for a living have come up with reasonably generic ways to detect problems in lookup tables, but I’m unaware of them. Notice that [MC/DC coverage](http://en.wikipedia.org/wiki/Modified_Condition/Decision_Coverage) — used in validating Level A software in DO-178B, and one of the stronger coverage metrics that is actually used in practice — is not any better than path coverage for finding the popcount bug.
