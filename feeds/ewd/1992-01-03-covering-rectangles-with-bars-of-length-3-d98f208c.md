---
title: "Covering rectangles with bars of length 3 (EWD1121)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1121.html
published: "1992-01-03T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1121.PDF
---

# Covering rectangles with bars of length 3

EWD 1121

Covering rectangles with bars of length 3.

In EWD1039 “Seemingly on a problem transmitted by Bengt Jonsson” I wrote on the following problem. Consider the 8×8 square; can it be covered by 21 bars of 3×1 and 1 little square of 1×1? Recently I heard a 3-dimensional variation: if a 4×4×4 cube is filled by 21 bars of 3×1×1 and 1 cube of 1×1×1, show that the little cube is situated in one of the corners of the big cube. In short, time to solve the problem in N dimensions.

I was struck by the inadequacy of the usual metaphors. Arguments about covering rectangles by 2×1 dominoes are traditionally carried out in terms of chess boards, and the above 8×8 problem was accordingly solved by colouring squares red, white, or blue. But we are used to painting surfaces and not to painting volumes. Another awkwardness is that in the one case we talk about “covering” and in the other case about “filling”. Even more seriously, our bars differ: from bars of 3×1 we went to bars of 3×1×1.

All these (linguistic) problems can be overcome by switching to an N-dimensional rectangular array of grid points or, equivalently, their N-tuples of coordinates. Covering and filling became partitioning, a bar becomes a triple with consecutive values in the only coordinate in which they differ. It now stands to reason to "colour" each point by 0, 1, or 2, viz. the sum of its coordinates reduced modulo 3.

If at least one of the lengths of the rectangle is a multiple of 3, partitioning in bars is obviously possible and, hence, the three colours occur with the same frequency. If none of the lengths of the rectangle is a multiple of 3, the total number of grid points is not a multiple of 3 and, hence, the colours do not occur with the same frequency. The frequencies, however, differ at most 1. (The proof is by mathematical induction over the number of dimensions. An (N+1)-dimensional rectangle with no length divisible by 3 can be turned into an (N+1)-dimensional rectangle with one length divisible by 3 by addition or removal of an N-dimensional rectangular slice with no length divisible by 3.)

With the total number of grid points equal to 3k � 1, we allow one “exceptional” point (of the colour that occurs k � 1 times): in the case 3k + 1, the exceptional point of a “partitioning” is the one that occurs in no bar —in the 4×1 rectangle, one of the end points is exceptional—, in the case 3k-1, the exceptional point of a “partitioning” is the one that is allowed to occur in two bars —in the 5×1 rectangle, the mid-point is exceptional—.

The possible positions of the exceptional point are further restricted by the consideration of symmetry that tells us that the colour of the exceptional point remains unchanged by reflection, i.e. if one coordinate, instead of numbering from m to n, is made to number from n to m. In particular:

a)
in a direction in which the length of the rectangle is of the form 3p + 1, the difference between extreme and exceptional coordinate is of the form 3q. (This deals with the 4×4×4 cube.)

b)in a direction in which the length of the rectangle is of the form 3p+1, the difference between extreme and exceptional coordinate is of form 3q+2. (This deals with the 8×8 square.)

For instance, in the 4×5 rectangle, one of the two marked positions occurs in two bars.

Nuenen, 3 January 1992

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
29-Nov-2011
