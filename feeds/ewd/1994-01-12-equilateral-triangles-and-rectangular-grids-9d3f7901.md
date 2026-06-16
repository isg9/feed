---
title: "Equilateral triangles and rectangular grids (EWD1170)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1170.html
published: "1994-01-12T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1170.PDF
---

# Equilateral triangles and rectangular grids

EWD 1170

Equilateral triangles and rectangular grids

The other day, I encountered the following problem: “Does there exist an equilateral triangle whose vertices have integer (orthogonal Cartesian) coordinates?”. The problem is, indeed, as elementary as it looks, but in solving it, I had a few pleasant surprises; hence this note. I urge the reader to think about the problem before reading on.

*         *         *

My first concern was how to characterize the notion “equilateral triangle”: all edges of the same length, all angles of the same size, or a mixture of the two. Considering that, in analytical geometry, lengths are more readily expressed than angles, I chose the first characterization, i.e. upon the triangle with with vertices (0,0), (a,b), (c,d) I imposed the requirement that its sides be of equal length. More precisely, my interest turned to integer quintuples (a,b,c,d,k) satisfying (0) ∧ (1) ∧ (2) with

(0)
a� + b� = k

(1)
c� + d� = k

(2)
(a� − c)� + (b� − d)� = k       .

Using (0) and (1), (2) can be simplified to

(2�)
2�(a�c + b�d) = k       .

Seeing that k is even, we turn our attention in view of (0) and (1) to squares and factors of 2. And here we have to make a little jump. It does not suffice to observe (the correct) even.x� ≡ even.x, which for a sum of 2 squares only implies even.(x�+y�) ≡ even.x ≡ even.y. We have to reduce squares modulo 4 and use

(3a)
x� mod 4 = 0   ≡   even.x

(3b)
x� mod 4 = 1   ≡   odd.x

from which we conclude about the sum of squares

(4a)
(x� + y�) mod 4 = 0   ≡   even.x ∧ even.y

(4b)
(x� + y�) mod 4 = 1   ≡   even.x ≢ even.y

(4c)
(x� + y�) mod 4 = 2   ≡   odd.x ∧ odd.y       .

And now we observe of an integer solution (a,b,c,d,k) of (0) ∧ (1) ∧ (2�):

k mod 4 ≠ 0

=         {(2�), in particular even.k}

k mod 4 = 2

=         {(0) and (1)}

(a�+b�) mod 4 = 2 ∧
(c�+d�) mod 4 = 2

=         {(4c) twice}

odd.a ∧ odd.b ∧ odd.c ∧ odd.d

⇒         {arithmetic}

even.(a�c + b�d)

=         {(2�)}

k mod 4 = 0       .

Having thus proved

(5)
k mod 4 = 0       ,

we conclude on account of (0), (1), (4a)

(6)
even.a   ∧
even.b   ∧
even.c   ∧
even.d       .

If (a, b, c, d, k) is an integer solution of (0) ∧ (1) ∧ (2), (a/2, b/2, c/2, d/2, k/4) obviously solves that equation as well; (5) and (6) tell us now that the latter is again an integer solution. By mathematical induction we conclude that any solution of (0) ∧ (1) ∧ (2) is divisible by any power of 2 (4), and hence equal to (0,0,0,0,0), 0 being the only integer with an unbounded number of divisors. All zeros is indeed a solution, i.e. there exist equilateral triangles whose vertices have integer coordinates, but such a triangle is degenerate and its vertices coincide.

*         *         *

For several reasons I was pleased with the above argument. Of course I regret the rabbit embodied by (3a)(3b), but I console myself with the thought that it is a fairly small one. The proof of k mod 4 = 0 is a nice example of proving P by constructing a weakening chain from ¬P to P, and the final part of the argument nicely exploits that only 0 has an unbounded number of divisors.

Remark   The latter observation can be turned into a heuristic: a good way of showing that a Diophantine equation has no other solution than 0, it suffices to show that any solution p can be divided by some n (n>1) such that p/n is again a solution. (End of Remark).

Inks Lake State Park,

12 January 1994

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
24-Dec-2011
