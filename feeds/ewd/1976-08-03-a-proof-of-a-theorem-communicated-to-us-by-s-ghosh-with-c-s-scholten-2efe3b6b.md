---
title: "A proof of a theorem communicated to us by S.Ghosh (with C.S.Scholten) (EWD582)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD582.html
published: "1976-08-03T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD582.PDF
---

# A proof of a theorem communicated to us by S.Ghosh (with C.S.Scholten)

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 582 A proof of a theorem communicated to us by S.Ghosh (with C.S.Scholten) :

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 233�234 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

A proof of a theorem communicated to us by S.Ghosh.
by Edsger W.Dijkstra and C.S.Scholten
In a letter of 19 August 1976, S.Ghosh (currently c/o Lehrstuhl
Informatik I, Universitat Dortmund, Western Germany) communicated without proof the
following theorem in natural numbers —here chosen to mean “nonnegative
integers”— :
Given a set of k linear equations of the form

Li = bi (0 ≤ i < k)(1)

in which the Li are homogeneous linear expressions in the unknowns with
natural coefficients and the bi are natural numbers, there exists a single
equation

M = c(2)

in which M is a homogeneous linear expression in the unknowns with natural
coefficients and c is a natural number, such that (2) has the same natural
solutions as (1).
Because the natural solutions of (1) are the common natural solutions of
(3) and (4), as given by

L0 = b0

L1 = b1 (3)

and
Li = bi for 2 ≤ 1 <(4)

it suffices to prove that (3) can be replaced by a single equation with the
same natural solutions as (3).
Consider for natural P0 and P1, to be chosen later, the equation

P0 * L0 + P1 * L1= P0 * b0 + P1 * b1(5)

All solutions of (3) are solutions of (5). We shall show that and P0 and P1
can be chosen in such a way, that, conversely, all natural solutions of (5)
are solutions of (3). We shall do so by choosing P0 and P1 in such a way
that (5), considered as an equation in L0 and L1 , has (3) as its only
natural solution; because all natural choices for the original unknowns will
give rise to natural L0 and L1 , this is sufficient.
Considered as an equation in L0 and L1 , the general parametric
solution of (5) is given by
L0 = b0 + t * P1
L1 = b1 - t * P0
(where, to start with, t need not be a natural number). We shall choose a
natural P0 and P1 in such a way that from natural L0 and L1 , viz.

b0 + t * P1 ≥ 0(6)

b1 - t * P0 ≥ 0(7)

left-hand sides of (6) and (7) integer(8)

we can conclude t = 0 .
Choosing P1 > b , we derive from (6)

t > -1(9)

Choosing P0 > b1 , we derive from (7)

t < 1(10)

Choosing P0 and P1 furthermore such that gcd(P0, P1) = 1 , we
derive From (8) that t must be integer; in view of (9) and (10) we conclude
that t = 0 holds. Summarizing: (5) can replace (3) provided

P0 > b1, P1 > b0 , gcd(P0, P1) = 1

*              *
*

Example. Let the given set be x = 1, y = 1, z = 1 . The first two equations
can be combined by choosing P0 = 2 and P1 = 3, yielding:

2*x + 3*y = 5 , z = 1 .

These two can be combined by choosing P0 = 2 and P1 = 7, yielding

4*x + 6*y + 7*z = 17

for which (1,1,1) is, indeed, the only natural solution. (End of example.)
3 August 1976

Plataanstraat 5

NL-4565 NUENEN

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-01-23

.
