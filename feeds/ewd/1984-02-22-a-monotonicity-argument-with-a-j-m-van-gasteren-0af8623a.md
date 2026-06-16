---
title: "A monotonicity argument (with A.J.M. van Gasteren) (EWD878)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD878.html
published: "1984-02-22T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD878.PDF
---

# A monotonicity argument (with A.J.M. van Gasteren)

AvG36 / EWD878

A monotonicity argument

In the following, square brackets denote universal quantification over
x and y ; f is a function of two arguments
such that

[x < y ≡> f
x y < f y x] .
(0)

From the above, we derive the following results.

Since p < q ≡ q > p
from (0):

[y > x ≡> f
y x > f x y] .

From the above, by renaming the dummies:

[x > y ≡> f
x y > f y x] .
(1)

From [x = y ≡> f x
y = f y x], by taking the term-wise
disjunction with (0) and and (1) respectively:

[x ≤ y ≡> f
x y ≤ f y x]
(2a)

[x ≥ y ≡> f
x y ≥ f y
x] . (3a)

By taking the counterpositives of (1) and (0) respectively —which to all
intents and purposes amounts to doing nothing—

[f x y ≤ f
y x ≡> x ≤ y]
(2b)

[f x y ≥ f
y x ≡> x ≥
y] . (3b)

From (2) and (3) respectively:

[x ≤ y ≡ f
x y ≤ f y x]
(4)

[x ≥ y ≡ f
x y ≥ f y
x] . (5)

From (4) and (5), by taking the term-wise conjunction:

[x = y ≡ f x
y = f y x] (6)

By negating both sides of (5) and (4) respectively:

[x < y ≡ f
x y < f y x]
(7)

[x > y ≡ f
x y > f y
x] . (8)

It is nice to see how the five equivalences (4) through (8) follow from the
single implication (0). Note that, since for reasons of symmetry (0) and (1)
are equivalent, all formulae could also have been derived from (1).

We draw attention to the fact that we did not use that f depends on
both its arguments: if f only depends on its first argument, (0)
postulates that f is an increasing function, but if f
only depends on its second argument, (0) postulates that f is a
decreasing function of that argument.

*      *

*

The above grew out of our dissatisfaction with the way we were introduced to
Euclidean geometry. With congruences, two theorems were proved seperately,
viz.

“An isosceles triangle has two equal angles.” and

“A triangle with two equal angles is isosceles.”

Prior to that it had already been shown that opposite to the larger angle lies
the larger side, in the jargon: α > β ≡> a
> b. This is a statement of the form (1); for the proofs of the
theorems, congruences are not needed at all, since the theorems are subsumed in
the conclusion corresponding to (6): α = β ≡ a =
b.

We had similar experiences with the two theorems

“An isosceles triangle has two angle bisectors of equal
length.” and

“A triangle with two angle bisectors of equal length is
isosceles.”

Thanks to the above, both theorems are subsumed in

“In a triangle, the larger angle has the shorter bisector.”,

a theorem that can be proved in a variety of ways. For instance, in a triangle
with sides of lengths a, b, and c, the square
of the length of the bisector of the angle opposite to a equals

b · c · (1 – (a / (b
+ c))2) .

This expression is an increasing function of b and a decreasing
function of a. It is not difficult to show that an f that
is an increasing function of its first argument and a decreasing function of
its second argument satisfies (0).

drs. A.J.M. van Gasteren

BP Venture Research Fellow

Dept. of Mathematics and

Computing Science

University of Technology

5600 MB EINDHOVEN

The Netherlands

22 February 1984

prof. dr. Edsger W. Dijkstra

Burroughs Research Fellow

Plataanstraat 5

5671 AL NUENEN

The Netherlands

transcribed by Matthew Hill

revised

08-Jan-2015
