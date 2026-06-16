---
title: "Another look at a problem from Hugo Steinhaus (EWD1311)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1311.html
published: "2001-10-16T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1311.PDF
---

# Another look at a problem from Hugo Steinhaus

EWD 1311

Another look at a problem from Hugo Steinhaus

Let a,b,c,d be, in order, the 4 sides of a convex polygon; let a and b meet at an end point of diagonal x, and let y be the other diagonal. If among the 3 edges meeting at a vertex, 1 is shorter than the other 2, that shortest one is marked. We have to show that at least one of the diagonals remains unmarked, i.e. we have to show

(0)
[(a ≤ x ∨ b ≤ x) ∧ (c ≤ x ∨ d ≤ x)]   ∨

[(b ≤ y ∨ c ≤ y) ∧ (d ≤ y ∨ a ≤ y)]

The first line expresses that x is unmarked; it is a conjuction of 2 conditions, 1 for each of its end points. The second line similarly covers y. Please note that in demonstrandum (0), each diagonal is compared with each side of our polygon.

To connect lengths of sides and diagonals with the convexity, we appeal to
the

Theorem 0  In a convex quadrilateral, the sum of the
lengths of the diagonals is at least the sum of the lengths of two opposite
sides.

(This theorem can be proved by appealing twice to the triangular
inequality and using that convexity means that the diagonals intersect each other between their end points.) We propose to show that (0) follows from

(1)
a + c ≤ x + y   ∧  b + d ≤ x + y.

As you see, we have not chosen a pair of opposite sides, we have taken
them both, mainly because the 4 sides all occur in (0), but also because, in this early stage, I did not want to destroy the symmetry. Finally there is a fair chance that we need both conjucts of (1) since a + c ≤ x + y by itself does not imply that x and y are the diagonals of a convex quadrilateral.

Before continuing with (1), we
rewrite (0): by distributing the outer ∨ over the
∧, we can express (0) as a conjuction of four terms,
and it is always nice to have your demonstrandum as a conjuction because one
can deal with conjucts separately. By distribution of the outer ∨ we
rewrite our demonstrandum (0) as

(2)
( a ≤ x ∨ b ≤ x ∨ b ≤ y ∨ c ≤ y ) ∧

( c ≤ x ∨ d ≤ x ∨ d ≤ y ∨ a ≤ y ) ∧

( a ≤ x ∨ b ≤ x ∨ d ≤ y ∨ a ≤ y ) ∧
( c ≤ x ∨ d ≤ x ∨ b ≤ y ∨ c ≤ y )

We now have to show
(1) ⇒ (2). We do so by eliminating to begin with the + sign, using

Theorem 1       p + q  ≤  r + s   ⇒   p ≤ r  ∨  q ≤ s.

(By taking the contrapositive, one recognizes a monoticity property of
addition.)

(1)

⇒    {Theorem 1, 4 times}

( a ≤ x ∨ c ≤ y ) ∧

( c ≤ x ∨ a ≤ y ) ∧

( b ≤ x ∨ d ≤ y ) ∧

( d ≤ x ∨ b ≤ y )

⇒    {termwise implication; ∧ monotonic}

(2)

QED.

*                       *

*

I recorded the above proof so that we can compare
it with the argument by Hugo Steinhaus. [Note: Steinhaus draws only the line
segments that we called "marked" and he assumes all distances to be different
.] I quote:

"Let us suppose that the newly formed figure includes two intersecting segments AB and CD.

Let us suppose that the points A and B have been connected by a segment because B lies nearest to A; and let us suppose similarly that the point D lies nearest to point C. Then AB < AD, CD < CB, whence AB + CD < AB + CB, which contradicts the proposition that in a convex quadrangle the sum of the diagonals is greater than the sum of two opposite sides.

In this we have proved [...] the theorem."

My proof seems to repeat the same argument about four times, each time with a different permutation of the letters; I have not been able to avoid that repetition by exploiting the symmetries. By phrasing the argument as a reductio ad absurdum, Steinhaus could postpone the introduction of nomenclature and thus reduce the repetition somewhat.

For a completely different proof by Lyn Pierce, see EWD 1251.

Austin, 16 October 2001

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

transcribed by Adam Dariusz Szkoda

revised Wed, 14 Nov 2007
