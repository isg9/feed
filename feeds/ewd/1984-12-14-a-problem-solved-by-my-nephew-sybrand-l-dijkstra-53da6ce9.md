---
title: "A problem solved by my nephew Sybrand L. Dijkstra (EWD904)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD904.html
published: "1984-12-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD904.PDF
---

# A problem solved by my nephew Sybrand L. Dijkstra

EWD 904

A problem solved by my nephew Sybrand L. Dijkstra

Given a circular arrangement of 16 lights and one pushbutton. When the button is pushed the new state of each light depends as follows on the (old) states of that light and its clockwise neighbour:

- when they were both on or both off, the light is on in the new state;

- when one was on and the other was off, the light is off in the new state.

Prove that all lights are burning after the button has been pushed 16 or more times.

*                      *

*
Having studied "Een methode van programmeren" --"A method of programming"-- by W. H. J. Feijen and me, and hence being familiar with the symmetry and associativity of the equivalence, my young nephew solved this problem essentially by the following argument.

An arbitrary light is called "nr.0", and the clockwise neighbor of light nr.i is called light nr. i+1. With

B(i,j) ≡ "light nr. i is burning after the j-th push"

the behaviour of the system is given by

(0)     B(i,j+1) ≡ B(i,j) ≡ B(i+1, j)

In preparation of a proof by mathematical induction we observe:

(i) since 20 = 1, (0) equivales

B(i, j+20) ≡ B(i,j) ≡ B(i+20, j).                 (End of (i).)

(ii) From (the induction hypothesis)

(1)     B(i, j+2n) ≡ B(i,j) ≡ B(i+2n,j)

we derive, since 2n + 2n = 2n+1, by substituing i+2n for i

(2)     B(i+2n, j+2n) ≡ B(i+2n, j) ≡ B(i+2n+1, j)

and, similarly, by substituting j+2n for j

(3)     B(i, j+2n+1) ≡ B(i, j+2n) ≡ B(i+2n, j+2n)

Eliminating B(i+2n, j+2n) from (2) and (3) yields

(4)     B(i, j+2n+1) ≡ B(i, j+2n) ≡ B(i+2n,j) ≡ B(i+2n+1, j)

Eliminating B(i, j+2n) ≡ B(i+2n, j) from (1) and (4) yields

(5)     B(i, j+2n+1) = B(i,j) ≡ B(i+2n+1, j)

i. e. with n+1 substituted for n. (End of (ii).)

Combining observations (i) and (ii), we have proved (1) for all natural n, in particular for n=4:

(6)     B(i, j+16) ≡ B(i, j) ≡ B(i+16, j)

Since lights nr. i and i+16 are identical, B(i, j) ≡ B(i+16, j), hence, from (6), B(i, j+16) .

q.e.d.

The above proof is, all by itself, nice enough to be recorded. I am particularly delighted, for when the section on predicate calculus in our book kindles the above in the mind of a 15-year-old, the seed is worth to be sown.

Neunen, 14 December 1984

Prof. dr. Edsger W. Dijkstra

Dept. of Computer Sciences

The University of Texas at Austin

AUSTIN, TX 78712-1188

United States of America

transcribed by Michael Lugo

revised Tue, 27 Mar 2007
