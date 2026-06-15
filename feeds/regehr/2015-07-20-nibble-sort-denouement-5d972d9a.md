---
title: Nibble Sort Denouement
url: https://blog.regehr.org/archives/1242
published: "2015-07-20T16:49:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=1242
---

# Nibble Sort Denouement

Back in January my [nibble sort contest](https://blog.regehr.org/archives/1213) resulted in entries that dramatically exceeded my expectations. Since then I’ve been trying to write up a post explaining the various strategies that people used and since you don’t care about my excuses I won’t tell you them, but I never got it written. However! I want to point everyone to three great blog entries:

- Jethro Beekman [explains his approach](http://jbeekman.nl/blog/2015/01/nibble-sort/) based on a sorting network

- Jordan Rose [walks us through the series of optimizations that he implemented, and also explains the fastest non-SIMD entry](http://belkadan.com/blog/2015/05/Nibblesort/)
- Hans Wennborg [adds an explanation of Alexander Monakov’s tour de force](http://www.hanshq.net/nibble_sort.html) which can sort the nibbles in a 64-bit word in just over 1ns

The thing that I liked most about this contest is that the fastest entries ended up being way faster than I predicted — mostly because I didn’t understand how excellent the Haswell vector unit is. I’ve sent my standard programming contest prize (Amazon gift certificate covering the cost of a paper copy of Hacker’s Delight) to Alexander and Jeff (Jerome’s non-SIMD entry, alas, arrived a bit after the contest expired). Thanks again everyone!

Final note: as a compiler researcher I think these contest entries could be very useful as a case study in how variance among algorithms and among implementations of algorithms can lead to extreme performance differences. In the long run we need compilers that understand how to automatically switch between different versions such as these. Regular compiler technology won’t cut it, nor will (at present) program synthesis. But it’s a good goal.
