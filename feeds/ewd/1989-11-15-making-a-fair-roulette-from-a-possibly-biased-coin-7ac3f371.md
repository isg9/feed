---
title: "Making a fair roulette from a possibly biased coin (EWD1071)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1071.html
published: "1989-11-15T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1071.PDF
---

# Making a fair roulette from a possibly biased coin

Copyright Notice

The following manuscript,

EWD 1071: Making a fair roulette from a possibly biased coin

was published as

Information Processing Letters 36: 193, copyright � 1990.

It is reproduced here by permission of the publisher, Elsevier Science.

Single copies of the manuscript may be downloaded and printed for the reader�s personal research and study.

EWD 1071

Making a fair roulette from a possibly biased coin

We are given a possibly biased coin, that is, a coin that, when flipped, shows head or tail with the probabilities p and q respectively, with p+q = 1 but not necessarily p=q. We don't know the values of p and q, we only know that the coin remains “the same”, i.e., that p and q are constant in time. The problem is to design an experiment with 37 different outcomes of equal probability.

In a variable A of type “sequence of 37 faces”, we record 37 flippings of the coin; if all the faces in A are the same, we start afresh, otherwise we accept the value of A. This first stage of the design is inspired by (i) the consideration that the experiment has to fail (to terminate) if the coin is so extremely biased that one of the faces never shows up, and (ii) the fact that (Fermat) 237-2 is a multiple of 37.

With A thus determined, we consider the sequences   rot.k.A   for 0 ≤ k < 37, where   rot.k.A   is obtained by rotating A over k positions (to the left say). Because A is not periodic with period 1 and 37 is prime, the 37 sequences   rot.k.A   (0 ≤ k < 37) are all different. Moreover, since they all contain the same number of heads (and the same number of tails), they are 37 equally likely A-values. And this solves our problem but for a convention that establishes a 1–1 correspondence between those 37 sequences and the numbers from 0 to 36, e.g. the value of k for which   rot.k.A   is the lexical minimum, i.e., with A thus determined, we deliver   r   satisfying

0 ≤ r < 37   ∧   (∀k: 0 ≤ k < 37:   rot.r.A ≼ rot.r.A)   .

Remark   In the above analysis it is irrelevant that the coin has only 2 faces; with a possibly loaded die, one can proceed in exactly the same manner. Essential use has been made of the primality of 37, but the generalization to a composite number is not difficult: we can make a fair die out of a fair binary roulette and a fair ternary roulette. (End of Remark.)

Austin, 15 November 1989

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
27-Nov-2011
