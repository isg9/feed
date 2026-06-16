---
title: "My methodological blunder with grid polygons (EWD1027)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1027.html
published: "1988-08-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1027.PDF
---

# My methodological blunder with grid polygons

EWD 1027

My methodological blunder with grid polygons

A "grid polygon” is defined as a plane polygon whose vertices are the only grid points on its edges.

Lemma   Consider a grid triangle with 1 interior grid point; the interior point is the centre of gravity of the triangle.

I thought for a while and constructed

and

.

Asked about my progress, I showed these two, adding “but I hate the case analysis and have as yet not shown that it is exhaustive”. How right was I about my caveat! The above case analysis is not exhaustive —there are infinitely many such triangles— and, moreover, it is totally superfluous.

I constructed these examples with the intention of convincing myself that grid triangles with 1 interior grid point do indeed exist. A stupid exercise, for if they don’t exist, the theorem is vacuously correct; furthermore, it is notoriously ineffective to study a non-empty set by looking at some of its members.

So we start afresh. The antecedent is about a grid triangle with 1 interior grid point, and the consequent is about a triangle and its centre of gravity. Start at the least familiar side, which is the antecedent. What can we say about a grid triangle with 1 interior point? By connecting the interior point with the vertices, we partition the original triangle into three grid triangles without interior grid points? A simpler concept!

What can we say about a grid triangle without interior grid points? In a way, it should not be too big, but if we are at a loss as how to make this constraint more precise, we can look at the consequent: what can we say about the size of the three triangles a big triangle is partitioned into when the centre of gravity is connected to the three vertices? The three small triangles have the same area, and, conversely, when a triangle is thus partitioned into three triangles of equal area, the interior point is the centre of gravity.

Since a grid triangle cannot be too small either we make a leap of faith and try to prove our theorem by showing that all grid triangles without interior grid points have the same area. Looking at the simplest example

we conclude that that area would have to be �. This fractional value begs to be eliminated, for, after all, grid points are about integer values. Fortunately, each grid triangle is half of a grid parallelogram

and if the grid triangle is without interior grid point, so is the grid parallelogram. So we are left with the obligation to show that a grid parallelogram without interior grid points has an area 1. This is most briefly shown by tiling the plane by such parallelograms, thus establishing a one-to-one correspondence between parallelograms and grid points — say, each parallelogram and its top-right vertex.

I am endebted to Wolfgang Hinderer (from Karlsruhe) for drawing my attention to this argument.

Nuenen, 23 August 1988

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

United States of America

transcribed by Corrado Cantelmi

revised
27-Nov-2011
