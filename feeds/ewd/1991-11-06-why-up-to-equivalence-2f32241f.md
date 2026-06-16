---
title: "Why “up to equivalence” (EWD1112)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1112.html
published: "1991-11-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1112.PDF
---

# Why “up to equivalence”

EWD 1112

Why "up to equivalence"

Let ⊑ be a preorder, i.e. a reflexive, transitive relation. Then relation ⋍, given by

p ⋍ q   ≡  p ⊑ q  ∧  q ⊑ p    ,

is reflexive, transitive and symmetric, i.e. it is an equivalence relation. In category theory, one encounters all the time conclusions valid "up to equivalence", a caveat that acts as a reminder that ⋍ is weaker than =   .  It makes one wonder why ⊑ is not strengthened into a partial order, i.e. is postulated to be anti-symmetric as well:

p = q  ⇐  p ⊑ q  ∧  q ⊑ p .

The following consideration gives a possible explanation.

*                     *

*

In the usual model of our predicate calculus, our atoms —P, Q, R, say— stand for boolean expressions in some variable, x say, and when we write

[P ≡ Q ∧ R ]      ,

the square brackets denote universal quantification over x.  For the purpose of this discussion I shall tag the square brackets with the name of the variable(s) quantified over;  the above is then rewritten as

[P ≡ Q ∧ R ]x

Let now x' be a fresh variable of some non-empty type. Then, for instance,

[P ≡ Q ∧ R ]x  ≡  [P ≡ Q ∧ R ]x, x'

since quantification over a fresh variable of a nonempty type is the identity operation. In other words, in the usual model of our predicate calculus we are free to view our atoms as expressions in a fresh variable as well, provided we reinterpret the square brackets accordingly. We try to capture this nice property by saying that our usual model of the predicate calculus is "insensitive to Cartesian extension".

Let us now investigate to what extent the usual model of the relational calculus is sensitive to Cartesian extension. The relational calculus is predicate calculus enriched with a unary operator called "transposition" (~), a binary operator called "composition" (;) and a constant (J) which  is composition's identity element. In the usual model for the relational calculus, the atoms are viewed as boolean expressions of two variables of the same (nonempty, but I am not going to repeat that all the time) type; we shall begin by denoting them by x and y respectively. That transposition is an involution we can express by

[P  ≡  ~~P ]x, y  ;

if you think it helpful, you can view the underlying space as a Cartesian square.

The transposition ~ is in the model

[~P  ≡  (x,y := y,x).P ]x,y   ,

and for this operation, the model is insensitive to Cartesian extension, since for fresh x', y'

[(x,x',y,y' := y,y',x,x' ).P   ≡  (x,y := y,x).P ]x,x',y,y'.

The composition  ;  is in the model

[P ;Q ≡〈∃z :: (y:=z).P ∧ (x:=z).Q〉]x,y    ,

and also for composition, the model is insensitive to Cartesian extension, since for fresh x', y'

[〈∃z :: (y:=z).P ∧ (x:=z).Q〉  ≡

〈∃z,z' :: (y,y':=z,z').P ∧ (x,x':=z,z').Q〉]x,x',y,y'.

Since "R is transitive" is formally expressed by

[R ;R  ⇒  R ]

the notion of transitivity is insensitive to Cartesian extension.

With the constant J, however, things are different. It is "the diagonal" x=y of the underlying Cartesian square; after extension with the fresh variables x',y', it is x=y ∧ x'=y'  . For fresh variables x',y' we observe

[ J ⇒ R ]x,x',y,y'

=         { extended def. of J }

[x=y ∧ x'=y' ⇒ R ]x,x',y,y'

=         { pred. calc. }

[[x=y ⇒ R ∨ (x'≠y' )]x',y']x,y

=         {x',y' are fresh}

[x=y ⇒ R ∨ [x'≠y' ]x',y']x,y

=         {x',y' of nonempty type}

[x=y ⇒ R ]x,y

=         { def of J }

[ J ⇒ R ]x,y

Since "R is reflexive" is formally expressed by [ J ⇒ R ]     , the above shows that also the notion of reflexivity, and hence the notion of a preorder, is insensitive to Cartesian extension.

The formal expression of "R is antisymmetric" is [R ∧ ~R  ⇒  J ]. This, however, is not insensitive to Cartesian extension, as is shown by the following observation for fresh variables x',y' (which we take from a nontrivial type, i.e. with at least 2 distinct values)

[R ∧ ~R ⇒ J ]x,x',y,y'

=         { extended def. of J }

[R ∧ ~R ⇒ x=y ∧ x'=y' ]x,x',y,y'

=         {pred. calc., x',y' fresh}

[R ∧ ~R ⇒ x=y ]x,y ∧ [x'=y']x',y'

=         {def. of J; fresh type nontrivial}

[R ∧ ~R ⇒ J ]x,y ∧ false

=         {pred. calc.}

false   ;

in other words: Cartesian extension destroys antisymmetry and turns a partial order into a preorder.

Austin, 6 November 1991

prof.dr Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 – 1188

USA

Transcribed by Guy Haworth.

Last revised on Sun, 6 Jan 2008.
