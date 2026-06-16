---
title: "The GCD and the minimum (EWD1313)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1313.html
published: "2001-11-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1313.PDF
---

# The GCD and the minimum

It all began with a friend who was preparing his undergraduate lectures asking me whether I had a nice calculational proof of

(0)
x↓y = 1   ⇒   x↓(y *z)  =  x↓z

(All variables are of type natural and ↓ stands for the greatest common divisor.) I did not have a nice proof of (0), so I started to think about it, and then the fun started. Hence this little note.

*               *

*

Having learned to like Lattice Theory and the Galois Connection, I immediately decided to regard ↓ as the infimum with respect to the partial order ⊑, read as "divides". The greatest common divisor ↓ is then defined by

(1)
u ⊑ x↓y   ≡   u ⊑ x  ∧  u ⊑ y
for all x,y,u.

From the above definition, many nice properties of ↓ elegantly follow, such as

(2)
↓ is associative, symmetric and idempotent

(3)
x↓y ⊑ x   and   x = x↓y  ≡  x ⊑ y

As the above are general results from elementary lattice theory and have nothing to do with integer arithmetic, I decided to use them freely as the need would arise. [It turned out that, to begin with, that need would only arise for (2).]

Besides ↓, (0) contains the special constants 1 and *, and the next question was how to capture their relevant properties. For 1 I could come up with two relations, connecting 1 to * and to ↓ respectively,

(4)
1*u = u
for all u, and

(5)
1↓u = 1
for all u,

and I postponed the choice.

Note At the time I did not realize that (4) is the more promising candidate. By presenting 1 as a unit element (of *), it offers a way of eliminating 1, while (5), which presents 1 as a zero element (of ↓), does not offer that service. (End of Note.)

(4) connects * with 1 but I assumed that this was not enough to capture the "relevant properties" of *. To connect * with ↓, I came up with

(6)
x↓(y *z)  =  x↓(x↓y * x↓z)
.

Now the proof of (0) was straightforward: we observe for any x,y,z (satisfying (6)

and (0)'s antecedent)

x↓(y *z)

=

{ (6) }

x↓(x↓y * x↓z)

=

{ antecedent of (0) }

x↓(1 * x↓z)

=

{ (4) with u := x↓z }

x↓(x↓z)

=

{ ↓ associative and idempotent }

x↓z   ,

which completes the proof of (0).

*               *

*

The above proof is nice, but one can object that it does not prove very much, as (6), which we use, is hardly simpler than the demonstrandum (0). [It is, for (6) does not contain 1.] So I started to think about proving (6), but in order to simplify matters, I switched to the additive version

(7)
x↓(y +z)  =  x↓(x↓y + x↓z)
,

where x,y,z are of type real and ↓ denotes the minimum. For this proof I used — besides (1) with ⊑ specialized as≤ —

(8)
x↓y ≤ u   ≡   x≤u  ∨  y≤u
for all x,y,u

which holds because ≤ is a total order — i.e. x≤y  ∨  y≤x — and

(9)
u + x↓y  =  (u+x)↓(u+y)
for all x,y,u

— i.e. + distributes over ↓ —, which follows from (1).

In order to prove (7) we observe

x↓(y+z)  =  x↓(x↓y + x↓z)

≡

{ + over ↓, i.e. (9) thrice }

x↓(y+z)  =  x↓(y+z)↓(x+x)↓(x+y)↓(x+z)

≡

{ (3) }

x↓(y+z)  ≤  (x+x)↓(x+y)↓(x+z)
(*)

≡

{ (8) }

x  ≤  (x+x)↓(x+y)↓(x+z)  ∨

(y+z)  ≤  (x+x)↓(x+y)↓(x+z)

≡

{ (1), 2 times twice }

( x ≤ x+x  ∧  x ≤ x+y  ∧  x ≤ x+z )   ∨

( y+z ≤ x+x  ∧  y+z ≤ x+y  ∧  y+z ≤ x+z )

≡

{ simplifications }

( 0 ≤ x  ∧  0 ≤ y  ∧  0 ≤ z )   ∨

( z ≤ x  ∧  y ≤ x )

We failed to prove (7) because it is not a theorem: it only holds for nonnegative x,y,z or when x is the largest of the three.

*               *

*

From (*) we first eliminated ↓ at the left-hand side, using (8), and then we used (1) to eliminate the ↓s at the right. We could have done it in the other order. We illustrate the consequences with a simpler example. Consider the calculations

x↓y  ≤  a↓b

≡

{ (8) }

x  ≤  a↓b   ∨   y  ≤  a↓b

≡

{ (1) }

(x ≤ a  ∧  x ≤ b)   ∨   (y ≤ a  ∧  y ≤ b)

and

x↓y  ≤  a↓b

≡

{ (1) }

x↓y  ≤  a   ∧   x↓y  ≤  b

≡

{ (8) }

(x ≤ a  ∨  y ≤ a)   ∧   (x ≤ b  ∨  y ≤ b)
,

from which we conclude

(x ≤ a  ∧  x ≤ b)   ∨   (y ≤ a  ∧  y ≤ b)   ≡

(x ≤ a  ∨  y ≤ a)   ∧   (x ≤ b  ∨  y ≤ b)

a beautiful formula that I did not know, but don't try to prove it by predicate calculus alone, for you need that ≤ is a total order.

Because, in contrast to ≤, ⊑ in the meaning of "divides" is not a total order we can expect (6) to be essentially harder to prove than its additive version (7). I would not be amazed if the uniqueness of the prime factorization were needed.

Acknowledgement I thank Hamilton Richards for starting me on these investigations and Jayadev Misra for his interest shown.

Austin, 27 November 2001

Prof. Dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

Transcribed by John S. Adair

Copy Editor: Ham Richards

Last revised on Wed, 14 Nov 2007.
