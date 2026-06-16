---
title: "Indirect equality enriched (and a proof by Netty) (EWD1315)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1315.html
published: "2001-12-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1315.PDF
---

# Indirect equality enriched (and a proof by Netty)

Proofs known as "proofs by indirect equality" traditionally exploit

(0)      x = y     ≡   〈∀u : true : u⊑x ≡ u⊑y 〉

for some reflexive, antisymmetric ⊑ : they establish equality by establishing the right-hand side of (0). The following lemma shows that we may be able to get away with a proof obligation that is formally weaker.

Lemma For reflexive, antisymmetric ⊑ and predicate P such that P.x ∧ P.y , we have

(1)      x = y     ≡   〈∀u : P.u : u⊑x  ≡  u⊑y 〉

Proof

LHS ⇒ RHS   This follows from Leibniz's Principle.

RHS ⇐ LHS   We observe for any x,y, P such that P.x ∧ P.y

〈∀u : P.u : u⊑x ≡ u⊑y 〉

⇒       { instantiate u:= x and u:= y }

(P.x ⇒ (x⊑x ≡ x⊑y )) ∧ (P.y ⇒ (y⊑x ≡ y⊑y ))

≡       { P.x ∧ P.y }

(x⊑x ≡ x⊑y ) ∧ (y⊑x ≡ y⊑y )

≡       { ⊑ is reflexive }

x⊑y ∧ y⊑x

⇒       { ⊑ is antisymmetric }

x = y                                           (End of Proof)

*           *

*

An application
Let the above ⊑ be the partial order of a lattice for which
the infimum ↓ is defined by the usual

(2)     u ⊑ x↓y    ≡    u⊑x  ∧  u⊑y

Fairly directly follow the general lattice properties

(3)     ↓ is idempotent, symmetric, associative

(4)     x↓y ⊑ x and x↓y ⊑ y

(5)     x⊑y   ≡   x = x↓y

We now specialize by making the variables x, y, u etc. of type natural and identifying ⊑ with "divides" —or "is a divisor of"—, which is a partial order on the naturals for which the infimum exists: x ↓ y is in fact the Greatest Common Divisor of x and y .

The formal link between our lattice and arithmetic on the naturals
—multiplication in particular— is given by providing 〈∃q :: q*x = y 〉 as a third expression for "x divides y ", i.e. we add to our laws

(6)     〈∃q :: q*x = y 〉   ≡   x⊑y

from which the mutually equivalent

(7)     m ⊑ m*x       and       m  =  m ↓ m*x

immediately follow. (Please note that we have given * a stronger binding power than ↓.)

We are now ready to prove that the GCD of two m-tiples‡ is an m-tiple‡, in formula

(8)     m  ⊑  m*x ↓ m*y

Proof    We observe

m   ⊑   m*x ↓ m*y

≡        { (5) and associativity of ↓ (3) }

m   =   m ↓ m*x ↓ m*y

≡        { (7) }

m  =  m ↓ m*y

≡        { (7) with x:= y }

true

(End of Proof)

‡ An "m-tiple" is a multiple of m.

And now we are ready for an application of the Lemma with which this note started: we shall show that multiplication distributes over the GCD, in formula

(9)     m * (x↓y )   =   m*x ↓ m*y

Proof

On account of (7), the LHS of (9) is an m-tiple, on account of (8), the RHS is an m-tiple. Lemma (1) can thus be applied with m ⊑ u for P.u . Accordingly we observe for 1 ≤ m —the case of m = 0 is obvious—

(9)

≡        { (1) }

〈∀u : m⊑u :  u ⊑ m*(x↓y )  ≡  u ⊑ m*x ↓ m*y 〉

≡        { transforming the dummy: u = m*w }

〈∀w :: m*w  ⊑  m*(x↓y )   ≡   m*w  ⊑  m*x ↓ m*y 〉

≡        { (2) }

〈∀w :: m*w ⊑ m*(x↓y )   ≡   m*w ⊑ m*x  ∧  m*w ⊑ m*y 〉

≡        { r ⊑s   ≡   m*r  ⊑  m*s for 1≤m }

〈∀w :: w ⊑ x↓y   ≡   w⊑x ∧ w⊑y 〉

≡        { (2) }

true

(End of Proof)

Addendum

For those who are not comfortable with the dummy transformation, here is its pattern in a bit more detail

〈∀u : m⊑u : Q.u 〉

≡        { for non-empty range 〈∀w :: C 〉≡ C }

〈∀u : m⊑u : 〈∀w : u = m*w : Q.u 〉〉

≡        { interchange of quantifications }

〈∀w :: 〈∀u : u = m*w  ∧  m⊑u : Q.u 〉〉

≡        { u = m*w  ⇒  m⊑u }

〈∀w :: 〈∀u : u = m*w : Q.u 〉〉

≡        { one-point rule }

〈∀w :: Q.(m*w ) 〉

(End of Addendum)

*           *

*

All of the above was triggered by Netty van Gasteren's use of (9) in her ingenious proof of

(10)     x↓y = 1  ⇒  x ↓ y*z = x↓z    :

x↓z

=        { antecedent of (10) }

x ↓ (x↓y )*z

=        { (9) and associativity of ↓ }

x ↓ x*z ↓ y*z

=        { (7) with m,x := x,z }

x ↓ y*z

Nuenen, 14 December 2001

prof. dr Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by David Naumann and Carl Ludwigson

revised Mon, 3 Dec 2007
