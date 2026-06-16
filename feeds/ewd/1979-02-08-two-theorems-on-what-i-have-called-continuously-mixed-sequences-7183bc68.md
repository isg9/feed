---
title: "Two theorems on (what I have called) continuously mixed sequences (EWD701)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD701.html
published: "1979-02-08T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD701.PDF
---

# Two theorems on (what I have called) continuously mixed sequences

EWD 701

Two theorems on (what I have called) continuously mixed sequences.

A sequence of characters from a finite alphabet is called "continuously mixed" if and only if for any position in the sequence each character from the alphabet occurs both somewhere on its left and somewhere on its right. As a result continuously mixed sequences extend infinitely, both to the left and to the right, and each character from the finite alphabet occurs infinitely often in such a continuously mixed sequence.

Lemma 1. A continuously mixed sequence from an N-character alphabet such that between any two successive occurrences of one character each other character occurs exactly only once, has a period of length N. (End of Lemma 1.)
Proof. The periodicity follows immediately from the observation that two successive occurrences of the same character are N positions apart. (End of Proof.)

Lemma 2. For continuously mixed sequences from a two-character alphabet we have

P0 or (P1 and P2)

with

P0:
between any two successive occurrences of one character the other character occurs exactly once

P1:
there exist successive occurrences of one character with less than one occurrence of the other character in between

P2:
there exist successive occurrences of one character with more than one occurrence of the other character in between. (End of Lemma 2.)

Proof. Lemma 2 follows from the observations non P0 = P1 and non P0 = P2 . (If the characters occur alternatingly, i.e. P0 holds, then neither P1 nor P2 holds; if the characters don't occur alternatingly, i.e. P0 doesn't hold, both P1 and P2 hold.) The equivalence P1 = P2 represents the surprise content of Lemma 2. (End of Proof.)

These two lemmata lead to

Theorem 1. A continuously mixed sequence from an N-character alphabet such that all successive occurrences of one character have at least one occurrence of each other character in between, has a period of length N . (End of Theorem 1.)

Theorem 2. A continuously mixed sequence from an N-character alphabet such that all successive occurrences of one character have at most one occurrence of each other character in between, has a period of length N . (End of Theorem 2.)

Proof. We focus our attention on an arbitrary pair of characters by removing all occurrences of all other characters. For the resulting continuously mixed sequence from a two-character alphabet, the antecedent of Theorem 1 is -- because its "at least one" is the negation of "less than one" -- equivalent to non P1 , and the antecedent of Theorem 2 is -- because its "at most one" is the negation of "more than one" -- equivalent to non P2 . In both cases, Lemma 2 establishes for the resulting continuously mixed sequences the truth of P0 . Because this conclusion holds for any pair of characters, the antecedent of Lemma 1 is satisfied, and its consequent proves the two theorems. (End of Proof.)

*                       *

*

The two theorema seem neither very deep nor very important. Since I discovered them last Friday, I have shown them to quite a few mathematicians -- you can do so without pencil and paper, over a cup of coffee -- and each time the theorems evoked surprise, and the proofs evoked amusement. I have not fathomed yet why, but in any case their reactions showed quite clearly that the above is "neat" enough to deserve being recorded.

8 February 1979

Plataanstraat 5

5671 AL NUENEN

The Netherlands
prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Sam Samshuijzen

revised Sun, 19 Dec 2004
