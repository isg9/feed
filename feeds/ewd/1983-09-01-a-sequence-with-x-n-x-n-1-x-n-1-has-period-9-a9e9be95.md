---
title: "A sequence with | x [ n ]| = x [ n –1] + x [ n +1] has period 9 (EWD862)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD862.html
published: "1983-09-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD862.PDF
---

# A sequence with | x [ n ]| = x [ n –1] + x [ n +1] has period 9

EWD862-0

A
sequence with |xn| = xn-1 + xn+1 has period 9

Recently I heard the
theorem that any (in both directions) infinite sequence of real numbers xn
such that for all n

|xn| = xn-1 + xn+1

(0)

has a period of
length 9. Here is my proof.

p

p + r

(≥
0)

r

(≥
0)

-p

(≤
0)

p - r

(≥
0)

2∙p -
r

(≥
0)

p

(≥
0)

r - p

(≤
0)

- r

(≤
0)

p

(≥
0)

p + r

From
(0) we conclude (i) that the sequence contains a nonnegative
element, (ii) that one of its neighbours is nonnegative, and (iii)
that at least one of the two elements adjacent to a pair of
nonnegative neighbours is nonnegative. More precisely: the
sequence contains in some direction a triple of adjacent elements
of the form (p, p+r, r) with 0 ≤
r ≤
p. To the left we have extended the sequence with another 8
elements. From (0) we further conclude that the whole sequence is
determined by a pair of adjacent values; hence, the repetition of
the pair (p, p+r) at distance 9 proves the
theorem. [The above deserves recording for its lack of case
analyses.]

Plataanstraat 5

5671 AL NUENEN

The Netherlands

1 September 1983

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcriber: Kevin Hely.

Last revised on Tue, 24 Jun 2003.
