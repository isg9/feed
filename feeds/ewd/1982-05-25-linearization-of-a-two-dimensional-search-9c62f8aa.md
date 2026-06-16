---
title: "Linearization of a two-dimensional search (EWD824)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD824.html
published: "1982-05-25T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD824.PDF
---

# Linearization of a two-dimensional search

EWD824-0

Linearization of a
two-dimensional search

Let the integer pair
(xi, yi) be denoted by Pi. Let of a finite number
of such pairs all xi be distinct and let all yi be
distinct. Let the pairs be partially ordered by Pi < Pj,
defined by

Pi < Pj
=  xi < xj  ∧ yi < yj

Pj is a minimal element means ¬(E i :: Pi < Pj),
and it is requested to find the minimal elements. We rewrite the
condition of minimality for Pj

¬(E
i :: Pi < Pj)

=  ¬(E i : xi < xj : yi
< yj)

=  (A i : xi < xj : yi
≥ yj)

Under the assumption
that the pairs have been numbered in the order of increasing x,
the above reduces to

(A i : i
< j : yi ≥
yj)  or  (A
i : i < j : yi > yj) .

In other words:
scanning the y's in the order of increasing x yields a new
minimal pair each time we encounter a y smaller than we have
encountered so far.

Since sorting can be
done in N ∙ log N steps, we have an N ∙ log N
algorithm. The solution is worth noting because I could not achieve that
result --to my great regret-- without destroying the symmetry between
the x's and the y's. The above was triggered by a
reconsideration of the longest upsequence problem.

Burroughs
Corporation

12201 Technology Blvd.

AUSTIN, Texas 78759

USA

25
May 1982

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcriber: Kevin Hely.
Last revised on Sat, 20 Sep 2003.
