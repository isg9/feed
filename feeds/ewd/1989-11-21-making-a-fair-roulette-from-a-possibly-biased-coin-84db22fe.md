---
title: "Making a fair roulette from a possibly biased coin (EWD1071a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1071a.html
published: "1989-11-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1071a.PDF
---

# Making a fair roulette from a possibly biased coin

Copyright Notice

The following manuscript,

EWD 1071: Making a fair roulette from a possibly biased coin

was published as

Information Processing Letters 36: 193, copyright � 1990.

It is reproduced here by permission of the publisher, Elsevier Science.

Single copies of the manuscript may be downloaded and printed for the reader�s personal research and study.

EWD 1071a

Making a fair roulette from a possibly biased coin.

We are given a possibly biased coin, i.e., a coin that, when flipped, shows head or tail with the probabilities p and q respectively, with p+q = 1, but not necessarily p=q. We don't know the values of p and q, we only know that the coin remains “the same”, i.e., that p and q are constant in time. The problem is to design an experiment with 37 different outcomes of equal probability.

In a variable B of type “sequence of 37 faces”, we record 37 successive flippings of the coin; if all the faces in B are the same, we start afresh, otherwise we accept the value of B. This first stage of the design has been inspired by (i) the consideration that the experiment has to fail (to terminate) if the coin is so extremely biased that one of the faces never shows up, and (ii) the fact that 237-2 is a multiple of 37. (One of Fermat's theorems states that for prime p, np-n is divisible by p.)

With B thus determined, we consider the 37 sequences obtained by rotating (to the left, say) for   0 ≤ k < 37   the sequence B over k positions. Because B is not periodic with period 1 and 37 is prime, these 37 sequences are all different. Moreover, since they all contain the same number of heads (and the same number of tails), they are 37 equally likely B-values. To complete our design, we have to introduce a convention that establishes a 1–1 correspondence between those 37 sequences and the numbers from 0 to 36. We can, for instance, associate with B the number of positions B has to be rotated over so as to yield the lexical minimum of those 37 sequences. (With 37 replaced by 2, the above yields a well-known method for making a fair coin from a possibly biased one.)

Remark   In the above analysis it is irrelevant that the coin has only 2 faces; with a possibly loaded die, one can proceed in exactly the same manner. Essential use has been made of the primality of 37, but the generalization to a composite number is not difficult: we can make a fair die out of a fair binary roulette and a fair ternary roulette. (End of Remark.)

Austin, 21 November 1989

transcribed by Corrado Cantelmi

revised
27-Nov-2011
