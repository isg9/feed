---
title: "From predicate transformers to predicates (Dedicated by the Tuesday Afternoon Club to C.A.R. Hoare at the occasion of his being elected Fellow of the Royal Society.) (EWD821)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD821.html
published: "1982-04-20T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD821.PDF
---

# From predicate transformers to predicates (Dedicated by the Tuesday Afternoon Club to C.A.R. Hoare at the occasion of his being elected Fellow of the Royal Society.)

From predicate transformers to predicates

Dedicated to the Tuesday Afternoon Club to C.A.R. Hoare at the occasion of his
being elected Fellow of the Royal Society.

Lemma 0. For any statement S and
any constant predicate C

wp(S, C) = wp(S,
T) ∧ C .

Proof. By substituting for C the
two constant predicates T and F, respectively. (End of
Proof.)

Lemma 1. For any statement S,
any predicate R, and any constant predicate C

wp(S, R ∨ C) =
wp(S, R) ∨ wp(S,
C) .

Proof. By substituting for C the
two constant predicates T and F, respectively. (End of
Proof.)

In the following, P is a predicate in x and by definition
P′ = Px′x ; variables x and
x′ range over the same non-empty domain.

Lemma 2. For any predicate P in
x we have for all x

P = (Ax′ ::
x ≠ x′ ∨
P′) .

Proof. P = (Ax′ :: P) = (Ax′ : x′ =
x : P′) = (Ax′ :: x ≠
x′ ∨ P′). (End of Proof.)

Theorem. For any statement S
with state space x and any predicate P we have

wp(S, P) = wp(S,
T) ∧ (Ax′ ::
wp(S, x ≠ x′)
∨ P′) .

Proof.  wp(S,
P)

= { Lemma 2 }

wp(S, (Ax′ :: x ≠
x′ ∨ P′))

= { distributivity of wp over universal quantification }

(Ax′ ::
wp(S, x ≠ x′
∨ P′))

= { Lemma 1 }

(Ax′ ::
wp(S, x ≠ x′)
∨ wp(S, P′))

= { Lemma 0 }

(Ax′ ::
wp(S, x ≠ x′)
∨ wp(S, T) ∧
P′)

= { wp(S, R) ⇒
wp(S, T) }

wp(S, T) ∧ (Ax′ :: wp(S,
x ≠ x′) ∨
P′) .

(End of Proof.)

Hence, the predicate transformer wp(S, ?) is fully
characterized by the two predicates wp(S, T)
and wp(S, x ≠
x′) .

20 April 1982

Edsger W. Dijkstra

Martin Rem

Ronald W. Bulterman

Jan L.A. van de Snepscheut

W.H.J. Feijen
Frans Peters

Jan Tijmen Udding

Jo Ebergen

A.J.M. van Gasteren

Maarten Boasson

transcribed by Matthew Hill

revised

11-Nov-2014
