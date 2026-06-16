---
title: "A (new?) proof of a theorem of Euler’s on partitions (EWD787)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD787.html
published: "1981-04-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD787.PDF
---

# A (new?) proof of a theorem of Euler’s on partitions

EWD787 - 0

A (new?) proof of a theorem of Euler's on partitions.

Euler's theorem. For any natural number N, the number of bags of (not necessarily distinct) odd natural nubers whose sum is N equals the number of sets of (distinct) positive integers whose sum is N.

Proof. In this proof:

i'sstand for positive integers,

q'sstand for odd natural numbers, and

t'sstand for powers of 2.

Euler's theorem follows from a 1-1 correspondence between bags of q's and sets of i's with the same sum.

In order to construct the bag of q's corresponding to a given set of i's, we observe that for each i the factorization i=t•q is unique, For each i in the set we put, with i=t•q, t instances of q into the bag. The result is a bag of q's with the same sum.

In order to construct the set of i's corresponding to a given bag of q's, we observe that each natural f is uniquely the sum of distinct t's. For a q with f occurrences in the bag we put into the set the i's of the form t•q for those distinct t's whose sum equals f. The result is a set of (distinct) i's

EWD787 - 1

with the same sum.

Grouping the i's in the set by largest odd divisor we see that the two transformations are each other's inverse. (End of Proof).

I found the above proof shortly after I had firmly decided to discard Ferrer diagrams and similar pictorial "aids". I asked myself how I could derive simply a bag of q's in a sum-preserving manner from a given set of i's. Since I was heading for a bag (in which multiple occurrences are allowed) I investigated how I could transform in a sum-preserving manner the individual i's into bags of q's. With the above result.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

27 April 1981

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

P.S. By distributing the above I learned that the proof is already known.

EWD.

Transcription by John C Gordon
Last revised on Sun, 13 Jul 2003.
