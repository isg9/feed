---
title: "Triggered by Wim Feijen’s treatment of “∃∀ ⇒ ∀∃” (EWD1201)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1201.html
published: "1995-02-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1201.PDF
---

# Triggered by Wim Feijen’s treatment of “∃∀ ⇒ ∀∃”

Triggered by Wim Feijen's treatment of "∃∀⇒∀∃" (EWD 1201)

In WF182, WJH Feijen presents totally convincing heuristics for the derivation of Carroll Morgan's proofs of the well-known theorem from the predicate calculus

(0)   [⟨∃ x :: ⟨∀ y :: t.x.y⟩⟩ ⇒ ⟨∀ y :: ⟨∃ x :: t.x.y⟩⟩] .

These proofs use the Laws of the Quantified Constant -- Wim Feijen regretted their anonymity, here is my suggestion for a name --

(1a)   [Q ⇒⟨∀ z :: Q⟩]   for fresh z

(1b)   [Q ⇐⟨∃ z :: Q⟩]   and any range ,

and the Laws of Instantiation, which are so well known that I won't write them down (now).

Reduced to its bare essentials, Carroll Morgan's proof is as follows

⟨∃ x :: ⟨∀ y :: t.x.y⟩⟩
⇒Quantified Constant
⟨∀ y' :: ⟨∃ x :: ⟨∀ y :: t.x.y⟩⟩⟩
⇒Instantiation y:=y'
⟨∀ y' :: ⟨∃ x :: t.x.y'⟩⟩

But, perhaps, this proof has been reduced a little bit too much, for we really did as if the unmentioned ranges were true, but in its full glory, the theorem to be proved is

(2)   [⟨∃ x: R.x: ⟨∀ y : S.y: t.x.y⟩⟩ ⇒ ⟨∀ y : S.y: ⟨∃ x : R.x: t.x.y⟩⟩] ,

and the Laws of Instantiation which I now give are, when we include the ranges, (for instance)

(3a)   [r.y ∧ ⟨∀ z: r.z: f.z⟩ ⇒ f.y]

(3b)   [f.y ⇒ (r.y ⇒ ⟨∃ z: r.z: f.z⟩)]

For Carroll Morgan's proof, with ranges included, I suggest the following text:

⟨∃ x : R.x: ⟨∀ y : S.y: t.x.y⟩⟩
⇒Quantified Constant
⟨∀ y' : S.y': ⟨∃ x : R.x: ⟨∀ y :: t.x.y⟩⟩⟩
=Range Diffusion, see below
⟨∀ y' : S.y': ⟨∃ x : R.x: S.y' ∧ ⟨∀ y :: t.x.y⟩⟩⟩
⇒Instantiation y:=y' , according to (3a)
⟨∀ y' : S.y': ⟨∃ x : R.x: t.x.y'⟩⟩

*
*
*

We observe for any punctual f and predicates Q,X,Y

[ Q ∧ f.X ≡ Q ∧ f.Y ] •
=pred. calc.
[ Q ⇒ f.X ≡ Q ⇒ f.Y ] •
=pred. calc.
[ Q ⇒ (f.X ≡ f.Y) ]
⇐f is punctual
[ Q ⇒ (X ≡ Y) ]
⇐pred. calc.
[ Y ≡ Q ∧ X ] ∨ [ Y ≡ Q ⇒ X ] •

Eliminating Y from the lines marked with a bullet, we conclude for punctual f

(4a)   [Q ∧ f.X   ≡   Q ∧ f.(Q∧X)]

(4b)   [Q ∧ f.X   ≡   Q ∧ f.(Q⇒X)]

(4c)   [Q ⇒ f.X   ≡   Q ⇒ f.(Q∧X)]

(4c)   [Q ⇒ f.X   ≡   Q ⇒ f.(Q⇒X)]

Note that the prefix Q∧ creates a predicate that depends monotonically on Q , whereas Q⇒ creates antimonotonic dependence.

A consequence of (4) --and trading-- is that the value of a universal or an existential quantification remains unchanged if we select a subexpression on which the term depends punctually and strengthen that subexpression by the prefix "range ∧" or weaken it by the prefix "range ⇒" . This property is referred to by "Range Diffusion".

In its applications it is, of course, useful to know that predicates built from ¬, ∨, ∧, ⇒, ⇐, ≡, ≢, ∀, ∃ depend punctually on their subexpressions (but since this has to be proved by induction over the syntax, I am afraid that Gries and Schneider would call this a metatheorem).

*
*
*

When we have integer dummies i,j with associated ranges 0≤i and 0≤j , we usually don't manipulate those ranges: we replace the ranges by (an invisible) true and declare the dummies to be of type natural number and allow this type information to diffuse all through their scope. That's how we do it.

Austin, 27 February 1995

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Dave Naumann

revised
21-Sep-2021
