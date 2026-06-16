---
title: "Proving Gupta’s Theorem (EWD989)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD989.html
published: "1986-10-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD989.PDF
---

# Proving Gupta’s Theorem

EWD 989

Proving Gupta’s Theorem

[actually de Ceva’s]

For an arbitrary triangle ABC a point P is chosen such that the points D, E, and F are well-defined as the above points of intersection. Gupta’s Theorem states

BD � CE � AF = −1        .

CD � AE � BF

The minus sign emerges by assigning to segments in opposite directions lengths of opposite sign and the theorem also holds for P outside the triangle. In the following we shall ignore this finesse and work with absolute values.

How do we prove this theorem? With due recognition of the symmetry, three ways seem to be open:

(i)   we try to prove

BD � CE � AF = CD � AE � BF

by independently dealing with both sides, i.e. “computing” them in such a fashion that their equality is apparent.

(ii)   we try to prove

(BD) � (CE) � (AF)  = 1

CDAEBF

by independently dealing with each of the three fractions, i.e. “computing” them in such a fashion that it is apparent that their product equals 1.

(iii)   we try to prove

(AF) � (BD) � (CE)  = 1

AEBFCD

by independently dealing with each of these three fractions.

Of these, (i) seems the least attractive: firstly, volumes are more complicated than dimensionless ratios and, secondly, in (i) the three-fold symmetry has not been factored out. Of the remaining two, (ii) is the most attractive, as a ratio of segments along the same line is “more dimensionless” than that of segments in different directions. (Here it shows that we made a mistake by immediately switching to absolute values: had we consciously kept our segments directed and our lengths signed, the definition of the signs in the fractions of (iii) would have caused trouble.) So we opt for (ii).

The three-fold symmetry having been factored out, it suffices to compute only one of the three fractions, say BD/CD. To begin with, we extract from our picture the part that defines that ratio by containing the two segments:

.

If nothing more has been given about these two segments, there is nothing more to be said about their ratio. Points B, D, and C being points of intersection, let us include in our figure those three intersecting lines:

.

This enables us to express our segment ratio as the ratio of the areas of two triangles, viz.

BD = BDA        .

CD  CDA

Staying with the above picture, there is little opportunity for rewriting the ratio of the two areas; we do observe, however, that the two triangles have the side AD in common. We include P in the picture and the lines BP and CP that divide the two triangles in the same ratio:

With our previous equality we now derive

BD = BPA      and      BD = BDP        .

CD  CPA  CD  CDP

The last equality is useless: it could have been deduced from

from which the information that we had 3 lines meeting in A has been lost. The first equality, however, does the job; this is nicely illustrated if we (i) remove the segment PD that is no longer needed and (ii) introduce E and F analogously to D :

*         *         *

The above was a sheer joy to write. (Yesterday evening I went to bed with an unfinished letter on my desk and I could have completed that letter. Instead, I followed Schubert’s example and composed an EWD after breakfast.)

I learned Gupta’s Theorem and the above proof from a book (co?)authored by Coxeter and remember being tickled by the elegance of the argument. Coxeter needed no more than half a page (picture included) and quite uninhibitedly pulled a few rabbits out of a hat. It was a pleasure to see how one rabbit after another could be exorcized, for that, of course, is what this whole exercise was about.

Gupta’s Theorem is of mild interest because to the best of my knowledge the Greeks missed it (which in all probability explains why to this very day it is still missing from almost all secondary school curricula in geometry!). And this Greek failure is more than a historical curiosity: it is a lesson. They paralyzed themselves by not treating products in their own right but forcing upon them the interpretation of areas and volumes.

Austin, 17 October 1986

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

United States of America

transcribed by Corrado Cantelmi

revised
15-Mar-2012
