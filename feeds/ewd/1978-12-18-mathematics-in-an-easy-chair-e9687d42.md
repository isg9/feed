---
title: "Mathematics in an easy chair (EWD695)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD695.html
published: "1978-12-18T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD695.PDF
---

# Mathematics in an easy chair

EWD695 - 0

Mathematics in an easy chair.

This afternoon W.H.J.Feijen posed me the following problem. Take a
circular piece of string -- of length 1, say -- and select three times a random
point of the string. Ignoring the case of coincident points -- which has a
probability = 0 -- we get three pieces of string when we cut the string at
the tree points chosen. How large is the probability that, stretched, these
three pieces could form a triangle, or, more precisely, if the lengths are
called x, y, and z respectively, how large is the probability that the
three inequalities x ≤ y + z, y ≤ z + x, and z ≤ x + y are satisfied?
Could I think of a simple argument that would not destroy the symmetry of the
problem statement?

I thought about the problem in an easy chair in the living room, while
enjoying a glass of sherry before dinner. The answer is 1/4, and here is
my argument.

To every choice of three points we can assign uniquely three values
x, y and z : we take them, for instance, in the clockwise direction,
starting with the first point chosen. Hence, for any value of z, the value
x is drawn homogeneously from 0 to 1 - z . For any set of values we
can plot in a three-dimensional space with three rectangular coordinates the
point (x, y, z); all these points lie in the plane with equation x + y + z = 1,
more precisely within the equilateral triangle with vertices (1,0,0), (0,1,0),
and (0,0,1) respectively. Because for fixed z --see above-- x is drawn
homogeneously from 0 to 1 - z , the probability density is in this triangle
constant along a line parallel to its side in the (x,y)-plane. For reasons
of symmetry the probability density is constant along a line parallel to any
of its three sides, and, hence, the probability density is constant over the
whole triangle. The required probability is therefore the area of the triangle
satisfying the three inequalities divided by the area of the whole triangle.
As the plane x = y + z cuts the triangle in the midpoints of two of its sides
--viz. (.5,.5,0) and (.5,0,.5)-- and the interior of the little triangle
corresponds to the points satisfying the triangular inequalities, the
probability sought equals 1/4.

EWD695 - 1

The above should not be interpreted as a recommendation for drinking
a glass of sherry before dinner. The circumstances under which I found the
above solution have, however been recorded as they form the N-th confirmation
(N large) of the experience that postponing the use of pencil and paper
is very helpful in finding a simple solution (if there is one, of course). If
I had a Thinking School, my students would have their hands tied behind their
back!

Finding the above solution took about ten minutes

18 December 1978

Plataanstraat 5

5671 AL Nuenen

The Netherlands

prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

Transcription by Joel Hockey
Last revised on Wed, 9 Jul 2003.
