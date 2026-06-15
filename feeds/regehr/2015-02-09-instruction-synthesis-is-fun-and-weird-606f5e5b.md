---
title: Instruction Synthesis is Fun and Weird
url: https://blog.regehr.org/archives/1219
published: "2015-02-09T23:22:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=1219
---

# Instruction Synthesis is Fun and Weird

*Synthesis* is sort of a hot topic in PL research lately. It basically means “implement a specification automatically.” Of course, at some level this isn’t very different from what compilers have been doing for ages, if we consider the source language program to be the specification, but when we’re doing synthesis the connotation is that the specification isn’t directly implementable using standard compiler techniques.

As of recently, we have an LLVM synthesizer for [Souper](https://github.com/google/souper). The basic method is from [this excellent PLDI paper](http://www.eecs.berkeley.edu/~jha/pubs/pldi-2011.pdf). In brief, you give the synthesis algorithm a specification and a library of components and it either gives you a way to wire together the components into a circuit that implements the specification or else it fails. Some other time I might explain the algorithm in detail, but here I wanted to give an example where we’re trying to figure out if a 64-bit word contains 16 repetitions of the same nibble value (this computation figured into many [nibble sort submissions](https://github.com/regehr/nibble-sort)).

The specification in C is:

```
int nibs_same(unsigned long word) {
  unsigned long n00 = (word >> 60) & 0xf;
  unsigned long n01 = (word >> 56) & 0xf;
  unsigned long n02 = (word >> 52) & 0xf;
  unsigned long n03 = (word >> 48) & 0xf;
  unsigned long n04 = (word >> 44) & 0xf;
  unsigned long n05 = (word >> 40) & 0xf;
  unsigned long n06 = (word >> 36) & 0xf;
  unsigned long n07 = (word >> 32) & 0xf;
  unsigned long n08 = (word >> 28) & 0xf;
  unsigned long n09 = (word >> 24) & 0xf;
  unsigned long n10 = (word >> 20) & 0xf;
  unsigned long n11 = (word >> 16) & 0xf;
  unsigned long n12 = (word >> 12) & 0xf;
  unsigned long n13 = (word >>  8) & 0xf;
  unsigned long n14 = (word >>  4) & 0xf;
  unsigned long n15 = (word >>  0) & 0xf;
  return
    (n00 == n01) & (n00 == n02) & (n00 == n03) & (n00 == n04) &
    (n00 == n05) & (n00 == n06) & (n00 == n07) & (n00 == n08) &
    (n00 == n09) & (n00 == n10) & (n00 == n11) & (n00 == n12) &
    (n00 == n13) & (n00 == n14) & (n00 == n15);
}

```

I won’t bother to give the LLVM version of this but it’s not too pretty, and obviously there’s not much that standard compiler optimization techniques can do to make it prettier. On the other hand, there’s a much better way to implement this specification:

```
int nibs_same(uint64_t word) {
  return word == rotate_right(word, 4);
}

```

LLVM doesn’t have a rotate instruction so the best we’ll do there is something like this:

**```** **define i32 @nibs_same(i64 %word) #0 {** **%1 = shl i64 %word, 60** **%2 = lshr i64 %word, 4** **%3 = or i64 %1, %2** **%4 = icmp eq i64 %3, %word** **%5 = zext i1 %4 to i32** **ret i32 %5** **}**

**```**

The question is: Will Souper generate this code from scratch? Turns out it does not, but on the other hand it generates this:

**```** **define i32 @nibs_same(i64 %word) #0 {** **%1 = lshr i64 %word, 28** **%2 = shl i64 %1, 32** **%3 = or i64 %1, %2** **%4 = icmp eq i64 %word, %3** **%5 = zext %4 to i32** **ret i32 %5** **}**

**```**

Or in C:

```
(((x >> 28) << 32) | (x >> 28)) == x

```

If you take a minute to look at this code, you’ll see that it is not implementing a rotate, but rather something odd and not obviously correct. It took me a while to convince myself that this is correct, but I believe that it is. The correctness argument uses these observations: (1) every nibble is compared to some other nibble and (2) it is irrelevant that the bitwise-or is presented with a pair of overlapping nibbles. Here’s the picture I drew to help:

**```** **0000000000111111111122222222223333333333444444444455555555556666** **0123456789012345678901234567890123456789012345678901234567890123** **>> 28** **000000000011111111112222222222333333** **012345678901234567890123456789012345** **<< 32** **00000011111111112222222222333333** **45678901234567890123456789012345** **|** **000000000011111111112222222222333333** **012345678901234567890123456789012345** **==?** **0000000000111111111122222222223333333333444444444455555555556666** **0123456789012345678901234567890123456789012345678901234567890123**

**```**

Also [Alive](https://github.com/nunoplopes/alive) can help:

**```** **%1x = shl i64 %word, 60** **%2x = lshr i64 %word, 4** **%3x = or i64 %1x, %2x** **%4x = icmp eq i64 %3x, %word** **%5 = zext i1 %4x to i32** **=>** **%1 = lshr i64 %word, 28** **%2 = shl i64 %1, 32** **%3 = or i64 %1, %2** **%4 = icmp eq i64 %word, %3** **%5 = zext %4 to i32**

**Done: 1** **Optimization is correct!**

**```**

The drawback of Souper's code is that since it doesn't include a rotate operation that can be recognized by a backend, the resulting assembly won't be as good as it could have been. Of course if LLVM had a rotate instruction in the first place, Souper would have used it. Additionally, if we posit a superoptimizer-based backend for LLVM, it would be able to find the rotate operation that is well-hidden in Souper's synthesized code.

I find it kind of magical and wonderful that a SAT solver can solve synthesis problems like this. Souper's synthesizer is very alpha and I don't recommend using it yet, but [the code is here](https://github.com/rsas/souper/compare/inst-synthesis#files_bucket) if you want to take a look. [Raimondas](http://www.cs.utah.edu/~rsas/), my heroic postdoc, did the implementation work.

I'll finish up by noting that my first blog entry was posted five years ago today. Happy birthday, blog.

**Update from Feb 10:** A reader sent this solution which I hadn't seen before:

```
(x & 0x0FFFFFFFFFFFFFFF) == (x >> 4)

```

I think Souper missed it since we didn't include an "and" in its bag of components for this particular run.
