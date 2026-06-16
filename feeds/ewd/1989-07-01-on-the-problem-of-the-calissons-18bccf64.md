---
title: "On the problem of the calissons (EWD1055)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1055.html
published: "1989-07-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1055.PDF
---

# On the problem of the calissons

EWD 1055

On the problem of the calissons

In the American Mathematical Monthly (May 1989, pp 429-431), and under the title “The Problem of the Calissons”, Guy David and Carlos Tomei present the following theorem.

Let a “calisson” be a diamond formed by two equilateral triangles with sides of length 1 that share a side. Let a regular hexagon with sides of length n be covered by (3n�) calissons; in such a covering the calissons occur in the three possible orientations in equal numbers.

In the following, a placed calisson is identified with a pair of adjacent equilateral triangles of the regular triangularization of the plane. Generalizing the hexagon, we call the set of triangles covered by a set of calissons a “box” that is “filled” by those calissons. A filling of a box has a “spectrum”, i.e., the triple of natural numbers counting the numbers of calissons in the three orientations. The theorem presented by David and Tomei is an immediate consequence of the more general

Theorem   All fillings of the same box have the same spectrum.

(Because of the rotational symmetry of the hexagon, the theorem tells us that the spectrum of a hexagon filling remains unchanged under cyclic permutation of its elements.)

To prove the theorem we consider an arbitrary box with two possibly different fillings. We partition the triangles of the box into the smallest subboxes that are common to both fillings; the theorem then follows from the fact that each subbox has the same spectrum in either filling, a fact we shall now demonstrate. In this demonstration we shall encounter two types of subboxes and for each type of subbox we shall argue separately that it has the same spectrum in either filling.

The one possible type of smallest common subbox is a pair of adjacent triangles: it corresponds to a calisson occurring in both fillings. Such a subbox admits only one filling, and hence all its fillings trivially have the same spectrum. This deals with any triangle that both fillings cover by the same calisson.

To investigate the other possible type of smallest common subbox, we consider a triangle that the two fillings cover by different calissons. For such a triangle we can identify two of its (three) adjacent triangles as its “neighbours” in the sense that they belong to the same subbox, the triangle being paired to the one neighbour in the one filling and to the other neighbour in the other filling, and each neighbour has again the property that the two fillings cover it by different calissons. The total number of triangles being finite, we conclude that the other possible type of smallest common subbox is a cyclic path of triangles such that neighbours on the path are adjacent.

We next observe that —in analogy to the chess board— the triangles of the regular triangularization can be coloured black and white, such that adjacent triangles differ in colour. This gives triangles in the same orientation the same colour and each calisson covers two differently coloured triangles. On the cyclic path the colours alternate, hence the cyclic path contains an even number of triangles and allows precisely two fillings: traversing the cyclic path in some direction, each black triangle is either paired to its white predecessor or to its white successor.

Consider now a grid line that cuts the cyclic path. Because the path is cyclic, the grid line cuts the path an even number of times and traversal of the path crosses the grid line as many times in the one direction as in the other direction, i.e., as many times from black to white as from white to black (triangles at one side of the grid line adjacent to it having the same orientation, and hence the same colour). Associating with each crossing the calisson that “sits astride” we now see that half of them belong to the one filling and half of them belong to the other filling. As all calissons astride the same grid line have the same orientation we conclude that the two possible fillings of a cyclic path have the same spectrum. And this observation concludes the proof.

*         *         *

David and Tomei give a very unsatisfactory treatment of this problem. I quote

“The idea of the proof is to reduce the problem to a very intuitive fact in three dimensions. We do not give complete details of a formal proof for two reasons. First, we do not want to spoil the simplicity of the basic intuitive idea. Furthermore, the result is an example of a proof by picture that does not translate immediately into precise mathematics.” Etc.

And then they come with an elaborate nonproof.

Their whole idea of interpreting the plane figure as a picture of a simple three-dimensional structure is, however, misguided as is shown when we generalize the hexagon to, for instance, the following ring-like structure:

M.C. Escher would be delighted.

I am indebted to the members of the Eindhoven Tuesday Afternoon Club, in particular to R.W. Bulterman for drawing our attention to the problem and to J.C.S.P. van der Woude for introducing the spectrum and conjecturing our theorem.

Nuenen, 5th of July 1989

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
27-Nov-2011
