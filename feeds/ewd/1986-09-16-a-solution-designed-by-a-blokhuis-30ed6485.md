---
title: "A solution designed by A. Blokhuis (EWD979)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD979.html
published: "1986-09-16T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD979.PDF
---

# A solution designed by A. Blokhuis

Consider all the chords between n distinct points on the perimeter of a circle, the points having been chosen such that each point inside the circle lies on at most 2 chords. Into how many regions is the area inside the circle divided by those chords?

The guess that for n = 5 the number equals 16 is correct; for n = 6, the number equals 31. (It is a very convincing example of the inadequacy of what is known as "Poor Man's Induction".)

*      *      *

The following argument derives an expression for the number of regions as a function of n.

In an effort of disentanglement we try not to use all our data simultaneously, in particular we temporarily ignore that eventually we will deal with all chords between n points. What can we say about the number of regions when a number of chords has been drawn?

Let us draw another chord:

Let
R = the number of regions

C = the number of chords

I = the number of intersections inside the circle

and let us investigate the relation between their increments for the case

ΔC = 1 ;

ΔR is the number of segments into which the new chord is divided by the others, which is one more than the number of its intersections, i.e.

ΔR = 1 + ΔI ;

from the above we derive

ΔR = ΔC + ΔI ;

and, since I = 0 and C = 0 corresponds to R = 1, by induction over C

R = 1 + C + I .

So we are done if we can determine C and I.
This is the moment to remember that we were considering n "peripheral" points and all chords between them.

Each chord determines two distinct peripheral points, and vice versa, hence

C = (n

2)

.

Each intersection, by determining two distinct chords, determines 4 distinct *) peripheral points, and vice versa **), hence

I = (n

4)

.

*) Intersecting each other already inside the circle, the two chords don't share a peripheral point.

**) For any 4 peripheral points, only 1 of the 3 possible pairings leads to chords intersecting inside the circle.

So

R = 1 + (

n

2
) + (
n

4
)

.

Since

(
n

k

) = (

n – 1

k – 1

) + (

n – 1

k
)

,   we can rewrite

R = 1 + (n – 1
1) +
(n – 1
2) +
(n – 1
3) +
(n – 1
4)

,

i.e. the sum of the first 5 entries of a line of the triangle of Pascal. The full lines adding up to the successive powers of 2, we now understand the misleading observations with which this note started.
The above argument is due to A. Blokhuis.

Austin, 16 September 1986

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 - 1188

United States of America

transcribed by Martijn van der Veen

revised Thu, 27 May 2010
