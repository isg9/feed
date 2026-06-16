---
title: "A proof by Rutger M.Dijkstra and me (EWD763)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD763.html
published: "1980-11-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD763.PDF
---

# A proof by Rutger M.Dijkstra and me

EWD763 -
0

A proof by Rutger M. Dijkstra and me.

Theorem. Consider the grid points, i.e. the points (x,y) with integer coordinates x and y, and let each grid point be painted with one of C distinct colours. For each X and Y, X distinct vertical grid lines and Y distinct horizontal grid lines exist such that their X.Y points of intersection have all the same colour.

Proof. Consider, for some k, the "strip" of points (x,y) with 0 ≤ x < k . In this strip the number of distinct possible colour patterns for a row is bounded (by Ck, to be precise). The number of rows in the strip being unbounded, at least one colour pattern occurs therefore in at least Y distinct rows. By choosing k larger than C.(X-1) we ensure that, in each and hence in this colour pattern, at least one colour occurs at least X times.

Acknowledgement. After having proved the theorem for X=Y=C=2 --that was the problem as originally posed-- my younger son, Rutger, removed during our discussion of the generalizations the last case analysis from the argument.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

23 November 1980

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Mon, 30 Jun 2003.
