---
title: "A.J.Martin’s solution of the Hungarian problem (EWD767)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD767.html
published: "1980-12-08T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD767.PDF
---

# A.J.Martin’s solution of the Hungarian problem

EWD767 -
0

A.J. Martin's solution of the Hungarian problem.

The day after I had shown EWD765 to Alain J. Martin, he told me a proof of Lemma 2 using mathematical induction. Lemma 2 was:

The game terminates or the difference between the largest and smallest value in the bag is unbounded.

Assume Lemma 2, which is true for the empty bag, to hold for a bag with N integers. Consider a non-terminating game for a bag with N+1 integers, and consider the two mutually exclusive cases

a)
there exists a value X that, from a certain moment, remains permanently in the bag; from that moment onwards the game is identical to the corresponding game with the N other integers.

b)
no value remains permanently in the bag; the largest value in the bag will increase and the smallest value will decrease, both beyond any bound. (Let, at a given moment X be the largest element in the bag and observe the state after the move in which X disappeared; for the conclusion it is irrelevant whether, prior to that move, X was still the maximum element.)

Plataanstraat 5

5671 AL NUENEN

The Netherlands

8 December 1980

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Mon, 30 Jun 2003.
