---
title: "An experiment in mathematical exposition (EWD731)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD731.html
published: "1980-02-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD731.PDF
---

# An experiment in mathematical exposition

EWD731 -
0

An experiment in mathematical exposition.

Many people feel attracted to the implication on account of the simplicity of the associated inference rules

A ⇒ B (1)
B ⇒ C
-----------
A ⇒ C

A ⇒ B (2)
C ⇒ D
-----------
A ∧ C ⇒ B ∧ D

A ⇒ B (3)
C ⇒ D
-----------
A ∨ C ⇒ B ∨ D

The transitivity of (1), and the symmetry of (2) and of (3) are clearly appealing. Rule (1), however, is a direct consequence of, and rules (2) and (3) are merely two different transcriptions of the same

A ∨ B (4)
C ∨ D
-------------
A ∨ C ∨ (B ∧ D) ,
a rule, which --on account of the symmetry of the disjunction-- can be applied in four different ways to the two given antecedents. Rules (2) and (3) give only two of the four. Rule (1) emerges as the special case

EWD731 -
1

A ∨ B
C ∨ ¬B
-------------
A ∨ C .

I called this "a direct consequence" because --perhaps somewhat arbitrarily-- I would like to distinguish between inference rules (different applications of which may yield results that are not equivalent) and simplifications that are possible according to boolean algebra --such as replacing B ∧ ¬B by false and A ∨ C ∨ false by A ∨ C-- , but never change the value of the boolean expression.

*

*

*

The above caused me to revisit the problem of the nine mathematicians visiting an international congress, and about whom we are invited to prove

A ∨ B ∨ C (5)

with

A:there exists a triple of mathematicians that is incommunicado (i.e. such that no two of them have a language in common)

B:there exists a mathematician mastering more than three languages

C:there exists a language mastered by at least three mathematicians.

Very much like the introduction of (named!) auxiliary lines or points in geometry proofs, I propose to introduce named auxiliary propositions, such that

EWD731 -
2

we can prove lemmata connecting them to the above propositions, such as

D:there exists a mathematician that can communicate with more others than he masters languages,

for which we can prove

Lemma 1 C ∨ ¬D.
Proof Obvious. With this qualification we mean here that we can start as well with observing
C ∨ "each mathematician communicates in different languages with those others he can communicate with", etc.

as with observing

¬D ∨ "there exists a mathematician that shares a language with at least two others", etc.

(End of proof of Lemma 1.)

With

E:there exists a mathematician that can communicate with more than three others,

we can prove

Lemma 2. A ∨ E.
Proof Let "x | y" here stand for "x and y are two different mathematicians that have no language in common. With
G:for each x, the equation x | u has at least five different solutions for u,

we observe (obviously)

E ∨ G (6)
With

EWD731 -
3

H:with y and z constrained to belong to an arbitrary quintuple, the equation y|z has at least one solution in y and z,

we observe (equally obviously)

E ∨ H . (7)
Applying rule (4) to assertions (6) and (7) we find     E ∨ (G ∧ H)   ,
hence
E ∨ "for each x, the equation x | y ∧ x | z ∧ y | z has at least one solution in y and z".

(End of proof of Lemma 2.)

Applying rule (4) to Lemmata 1 and 2 we infer the

Corollary.     A ∨ C ∨ (E ∧ ¬D)   .
Remembering rule (4) we see that (5) has been proved when we can prove B ∨ ¬(E ∧ ¬D)
or, equivalently

Lemma 3. B ∨ D ∨ ¬E.
Proof. Obvious. (End of proof of Lemma 3).

*

*

*

Note that in the above the Corollary was only used for heuristic purposes. Once Lemmata 1, 2, and 3 have been established we could have inferred

A ∨ E
B ∨ D ∨ ¬E
---------------
A ∨ B ∨ D

and

A ∨ B ∨ D
C ∨ ¬D
---------------
A ∨ B ∨ C

EWD731 -
4

and our two individual inferences would have been of the traditional form of the transitive implication.

*

*

*

I know that firm believers in the so-called "natural deduction" will state that, in the case of Lemma 2, I am just "deducing naturally" that A follows from the "assumption" ¬E. In this appreciation they will find themselves strengthened by the observation that in that proof all assertions start with "E ∨". They have a point, but the point is weak. Look at the structure of the proof as a whole. Lemmata 1, 2, and 3 capture it; from there rule (4) does the job, and at that level it is very arbitrary to subdivide assertions into assumptions and conclusions.

Remark. Observing the seven triples xyz for a pair (x,y) such that x | y , the argument proving Lemma 2 can equally well be phrased in terms of assertions starting with "A ∨". In the sense used above also Lemma 2 is obvious. (End of remark.)

Plataanstraat 5

5671 AL NUENEN

The Netherlands

27 February 1980

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Tue, 24 Jun 2003.
