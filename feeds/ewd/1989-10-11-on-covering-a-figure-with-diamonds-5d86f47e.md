---
title: "On covering a figure with diamonds (EWD1055c)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1055c.html
published: "1989-10-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1055c.PDF
---

# On covering a figure with diamonds

EWD 1055c

On covering a figure with diamonds

In the American Mathematical Monthly (May 1989, pp 429-431), under the title “The Problem of the Calissons”, Guy David and Carlos Tomei present the theorem given below. Before formulating the theorem, we introduce some terminology.

We consider a regular triangularization of the Euclidean plane, the grid lines of which cut up the plane into equilateral triangles with sides of length 1; in the following, “triangle” refers to such an equilateral triangle. A pair of triangles being “adjacent” means that they are different and share a side; such a pair of adjacent triangles is also referred to as a “diamond”, and a moment’s reflection tells us that each diamond has one of three possible orientations. A "figure” is a finite set of triangles; a “covering” of a figure is a partitioning of the figure’s triangles into diamonds.

The theorem presented by David and Tomei states that in any covering of a regular hexagon with sides of length n (and comprising 6n� triangles), the diamonds occur in the three orientations in equal numbers.

*         *         *

Because of the rotational symmetry of the hexagon, the above theorem is an immediate consequence of the following one.

Theorem   Consider a figure that admits one or more coverings. Each covering yields a triple of frequencies, each counting the diamonds in one of the considered yield the same triple.

To prove this theorem, we consider a figure that admits two possibly different coverings A and B, and show that A and B yield the same triple of frequencies. To this end we partition the triangles of the figure into two groups, (i) and (ii); note that each triangle of the figure belongs to a diamond in A and to a diamond in B.

(i)   The triangles belonging to the same diamond in both coverings. Their contribution to the A-triple is by definition equal to their contribution to the B-triple.

(ii)  The triangles belonging in A and B to two different diamonds. To investigate this latter case, we visualize for a moment the following graph: its vertices “are” the triangles in group (ii) and its edges “are” the diamonds in A and B. This graph is finite (because a figure was defined to be a finite set of triangles) and each vertex is connected to exactly two others (by edges corresponding to the diamonds to which the triangle belongs in A and B respectively). Graph theory tells us that such a graph falls apart into cycles. We conclude that the triangles in group (ii) are partitioned into cyclic paths of adjacent triangles, pairs of neighbours alternatingly forming diamonds from A and B. The graph we visualized can now be forgotten. We can now meet our proof obligation by proving the following lemma.

Lemma   Let a “cyclic” path be a set of triangles that can be arranged cyclically so that each pair of neighbours in the cycle is adjacent. A cyclic path admits two coverings, which yield the same triple of frequencies.

To prove this lemma, we observe that the regular triangularization contains triangles in only two orientations and that two adjacent triangles are of different orientation. In honour of the chessboard we call the triangles of the two orientations “black” and “white” respectively. (Like the domino on the chessboard, the diamond pairs fields of opposite colour.) That the cyclic path admits exactly two coverings is obvious, but now we can be more precise: in the cyclic path, colours alternate, and in the one covering each black triangle is paired with its white successor, while in the other covering each black triangle is paired with its white predecessor. It is this observation that gives the clue to the argument that both coverings yield the same triple of frequencies.

We demonstrate this sameness by showing how to establish a one-to-one correspondence between equally oriented diamonds from the two coverings of a cyclic path. Take an arbitrary diamond from one of the coverings and consider the grid line that separates the two triangles of that diamond: we say that, in that diamond, the cyclic path “crosses” that grid line. In a complete traversal of the cyclic path, that grid line is crossed as many times in the one direction as it is crossed in the opposite direction, that is: as many times from a black triangle to a white one as from a white triangle to a black one, that is: half of the diamonds in which the grid line is crossed come from the one covering of the cyclic path and half of them from the other. Furthermore, all diamonds thus halved by a grid line have the same orientation, and this concluded the proof.

*         *         *

I am indebted to the members of the Eindhoven Tuesday Afternoon Club, in particular to R.W. Bulterman for drawing our attention to the problem —David and Tomei had given an elaborate nonproof— and to J.C.S.P. van der Woude for conjecturing our more general theorem. I am indebted to the members of the Austin Tuesday Afternoon Club, in particular to Vijaya Ramachandran for simplifying my argument, to Jayadev Misra for suggesting improvements in the presentation, and to the graduate students for their invaluable assistance in the final editing of this note.

Austin, 11 October 1989

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
27-Nov-2011
