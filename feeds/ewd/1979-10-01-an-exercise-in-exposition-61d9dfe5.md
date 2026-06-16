---
title: "An exercise in exposition (EWD717)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD717.html
published: "1979-10-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD717.PDF
---

# An exercise in exposition

EWD717 -
0

An exercise in exposition.

The body of this note describes my solution to a problem communicated to me by R.W. Bulterman. The note is written because, after I had found my solution, it was not immediately clear how to describe it well. (This text is my second effort.)

The problem.

We consider a graph consisting of a pentagon A and a pentagon B, extended with all edges that connect a vertex from A with a vertex from B. (The graph has 2x5 = 10 vertices and 2x5 + 5x5 = 35 edges.) Furthermore it has been given that

1)each edge is either white or black, and

2)no three edges of the same colour form a triangle.

Prove that the five edges of pentagon A and the five edges of pentagon B have all ten the same colour. (End of problem.)

The proof.

Arrange the 25 edges that connect a vertex from A with a vertex from B in a 5x5 array, such that the cyclic order of the columns corresponds to that of the vertices of A, and the cyclic order of the rows corresponds to that of the vertices of B. There are five pairs of "adjacent columns" because the first and the last columns

EWD717 -
1

are also considered as an adjacent pair; similarly for the rows. (The array fits on a torus.)

We denote by "a cycle" either a row or a column. By calling two cycles "orthogonal" to each other, we denote that the one is a row and the other is a column. Two adjacent cycles are said to be "linked by a colour" if and only if there exists a cycle orthogonal to them in which they both contain an element of that same colour.

From 2) we can now conclude:

3)no two adjacent cycles are linked by both colours.

for if, for instance, two adjacent columns are linked by one colour, 2) allows us to conclude that the corresponding edge of A has the opposite colour.

The "dominant colour" of a cycle is defined as the colour occurring most frequently in it. (Because each cycle contains 5, i.e. an odd number of elements, the dominant colour of a cycle is always defined.)

Consider, for instance, a column for which the dominant colour is white; it then contains white elements in two adjacent rows. On

EWD717 -
2

account of 3), an adjacent column contains at least one white element in those two rows, i.e. the two adjacent columns are linked by white. In general we conclude:

4)adjacent cycles are linked by the dominant colour of any one of them.

On account of 3) and 4) we conclude:

5)adjacent cycles have the same dominant colour.

Hence --by induction--

6a)all columns have the same dominant colour

6b)all rows have the same dominant colour.

Because the dominant colour of the array as a whole equals the dominant colour of the columns, but also that of the rows, 4) allows us to conclude:

7)all ten pairs of adjacent cycles are linked by the dominant colour of the array as a whole,

and on account of 1), 2), and 7) we conclude

8)the colour of any edge from pentagon A or pentagon B is opposite to the dominant colour of the array as a whole.

(End of proof.)

Plataanstraat 5

5671 AL Nuenen

The Netherlands

3rd October 1979

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Tue, 24 Jun 2003.
