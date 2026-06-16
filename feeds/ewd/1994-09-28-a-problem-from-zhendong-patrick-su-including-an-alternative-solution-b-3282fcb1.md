---
title: "A problem from Zhendong Patrick Su (including an alternative solution by R.Boute (EWD1188)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1188.html
published: "1994-09-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1188.PDF
---

# A problem from Zhendong Patrick Su (including an alternative solution by R.Boute

EWD 1188

A problem from Zhendong Patrick Su

The other week, my student Zhendong Patrick Su showed me the following problem.

Consider the following quadrilateral with a diagonal, in which the sizes of the angles are indicated in units of π/36

We are asked to determine the angle marked below, and suggest that you think about the problem before turning the page.

One thing that strikes the reader is that one of the sides connects two equal angles (of 22 each), and the question is whether we can exploit that symmetry. The suggestion is to try to do that by introducing a symmetric trapezium (a quadrilateral with 1 pair of parallel sides)

Filling in the angle sizes, we see that the figure is very special indeed: the figure has been partitioned into a symmetric trapezium and an isosceles triangle.

The symmetry of the trapezium can be exploited by observing that as a result its 4 vertices lie on a circle; consequently the two marked angles are equal.

The isosceles triangle with top of size 22 is exploited by considering the circle through its other 2 vertices and with the top of size 22 as centre. Because of the angle marked 11, the 4th vertex lies on that circle (and because of the fact that the vertices of the angles of size 11 and 22, respectively, lie at the same side of the line connecting the other two vertices in the picture), we can thus mark a 3rd segment // and we have found 2 other isosceles triangles. Either of them suffices to conclude that the size of the marked angle, i.e. the requested answer, is 8 (36−2*14 or 36−2*3−22).

*         *         *

I thought that the very first step was a rather big rabbit, but while I was writing this note, my wife Ria tried to solve this problem, and the partitioning into a symmetric trapezium and an isosceles triangle was the very first thing she did!

That I used the theorem —proved in EWD1180— that a circle is the locus of points from which 2 given points are viewed under a constant angle, etc., is not surprising: it is a sophisticated packaging of the equal lines and the equal angles of the isosceles triangle.

I wrote this down for the sake of presentational experiment of not naming the components of the picture.

Austin, 30 September 1994

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

Supplement

An alternative solution to Zhendong Patrick Su’s problem

The solution proposed in EWD1188 exploits the symmetry induced by two specific angles of size 22.

The solution proposed here exploits the more direct symmetry with respect to the “major diagonal” bisecting one of these angles. This symmetry is completed by connecting the mirror image of the vertex with angle of size 21 to all vertices of the quadrilateral, as shown in the following picture:

The resulting isosceles triangle with top angle marked 11 + 11 obviously has base angle 18 − 11 = 17, as indicated. Since the angle marked 7 equals the angle marked 3 + 4, the four points highlighted by dots are on a circle. Hence the angle between the base of the isosceles triangle and the minor diagonal equals the angle marked 1. Therefore the angle marked ? has size 7 + 1 = 8.

Discussion. Consider the generalization of this problem where the given values 11, 3, 4 are replaced by arbitrary values α, β, γ respectively. The condition for the 4 dots being on a circle is 18 − α = β + γ, i.e. α + β + γ = 18. Clearly the unknown angle = (γ + β) + (γ − β) = 2 � γ, resulting in a triangle with angles 2 � α, 2 � β and 2 � γ. This generates a family of equivalent problems, even if we require that γ − β = 1 as in the original problem statement.

In EWD1188, the existence of the isosceles triangle used in the derivation requires 2 � α + (β + γ) + (β + γ) = 36, hence α + β + γ = 18 as before. The symmetry arguments based on the equality of two specific angles of the quadrilateral results in the extra condition α + 2 � α + β = 36, i.e. 3 � α + β = 36. If we also require γ − β = 1, then the only possibility is α, β, γ = 11, 3, 4.

R. Boute, 1995/01/24

transcribed by Corrado Cantelmi

revised
24-Dec-2011
