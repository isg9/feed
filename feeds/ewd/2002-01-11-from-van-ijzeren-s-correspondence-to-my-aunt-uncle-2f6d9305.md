---
title: "From van IJzeren’s correspondence to my aunt & uncle (EWD1317)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1317.html
published: "2002-01-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1317.PDF
---

# From van IJzeren’s correspondence to my aunt & uncle

From van IJzeren's correspondence to my aunt and uncle (EWD1317)
-

EWD1317

From van IJzeren's correspondence to my aunt and uncle

Browsing through letters sent to my late aunt dr H.A. van Herk-Kluyver, I found the following theorem and proof in a letter from my late brother-in-law mr dr J.van IJzeren. The theorem is surprising but not interesting; this note is devoted to its proof because it is an elementary and very convincing illustration of the use of the device of the General Solution (which I promoted in EWD1155).

The proof uses two theorems.

(0) A Pythagorean triple that is relatively prime, can be written as p2 – q2, 2pq, p2 + q2 (See EWD1172.)

(1) For prime p, np – n is a multiple of p. (Known as "Fermat's Little Theorem", see EWD740.)

The theorem to be proved here —as said: surprising but not interesting— is

(2) For any Pythagorean triple, i.e. with a2 + b2 = c2, there is a multiple of 7 among (a), (b), (a−b), (a+b).

*

*

*

Proof Because of a2 + b2 = c2, any factor shared by 2 of the numbers is shared by the 3rd, and hence we can restrict ourselves to a, b, c that are relatively prime. We shall do so in what follows.

With ⊑ denoting "divides" and p,q providing (see (0)) the parameters for the General Solution of a2 + b2 = c2, we observe

7 ⊑ a ∨ 7 ⊑ b ∨ 7 ⊑ a-b ∨ 7 ⊑ a+b

≡   { algebra, prime.7 }

7 ⊑ ab(a2 – b2)

≡   { a := p2 – q2, b := 2pq }

7 ⊑ 2(p2 – q2)pq(p4 - 2p2q2 + q4 – 4p2q2)

≡   { ¬ 7 ⊑ 2, prime.7, algebra }

7 ⊑ pq(p2 – q2) (p4 - 6p2q2 + q4)

≡   { modulo calculus }

7 ⊑ pq(p2 – q2) (p4 + p2q2 + q4)

≡   { algebra }

7 ⊑ pq(p6 – q6)

≡   { algebra }

7 ⊑ p7q – pq7

≡   { Fermat's Little Theorem, modulo calculus, prime.7 }

7 ⊑ pq – pq

≡   { 7 ⊑ 0 }

true.

I don't think I could have proved this theorem without the introduction of p and q.

Austin, 11 January 2002

prof.dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 – 1188 USA

Transcribed by Guy Haworth.

Last revised Wed, 14 Nov 2007.
