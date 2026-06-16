---
title: "Sylvester’s theorem used (see EWD1016) (EWD1228)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1228.html
published: "1996-01-13T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1228.PDF
---

# Sylvester’s theorem used (see EWD1016)

EWD 1228

Sylvester’s theorem used (see EWD1016)

On Tuesday 2 January 1996, Ronald W. Bulterman told me the following theorem.

Consider, for n ≥ 2, n distinct points in the plane; let k be the number of all distinct lines such that each of them contains at least 2 of those points. Then   k = 1 ∨ k ≥ n   .

*         *         *

Proof   The proof is by mathematical induction on n. There are two reasons for trying this proof structure, firstly the shape of the disjunct k ≥ n in the demonstrandum and secondly the fact that (Euclid’s Axiom)   n = 2 ⇒ k = 1   immediately establishes the base.

With   nʹ = n +1, we consider for the induction step nʹ distinct points, giving rise to kʹ  lines. We have to show   kʹ = 1 ∨ kʹ ≥ nʹ, where we may use k = 1 ∨ k ≥ n “ex hypothese”. In view of our demonstrandum being a disjunction, we introduce a case analysis, viz. kʹ =1 versus kʹ ≠1.

kʹ = 1

In this case, the demonstrandum   kʹ = 1 ∨ kʹ ≥ nʹ   follows directly (i.e. by predicate calculus alone).

kʹ ≠ 1

In this case, the demonstrandum   kʹ = 1 ∨ kʹ ≥ nʹ   simplifies directly to kʹ ≥ nʹ. The remainder of the proof is devoted to showing how, for nʹ noncollinear points, kʹ ≥ n follows from the induction hypothesis.

In order to be able to appeal to the induction hypothesis, we single out one of the nʹ points —let us call that point “A”— and consider the remaining n points and the k lines they give rise to all by themselves. Ex hypothese we may use k = 1 ∨ k ≥ n; we exploit the two disjuncts separately.

In the case k = 1, the n (distinct) points lie on a single line, and, because the nʹ points are noncollinear, that line does not contain A. Hence kʹ = n + 1, and since nʹ = n + 1, kʹ ≥ nʹ has been established.

In the remaining case k ≥ n —or, since nʹ = n + 1, equivalently k + 1 ≥ nʹ — , our demonstrandum kʹ ≥ nʹ follows from kʹ ≥ k + 1 or, equivalently, kʹ  > k. How do we establish kʹ  > k? Or, in other words, how can we conclude that the removal of A reduces the number of lines? Well, since a line has to go through at least 2 points of the set, such a line disappears if A is one of the only 2 points it goes through. Hence we can assert kʹ  > k by a proper choice of A provided:

“For any number of distinct, noncollinear points in the plane, there exists a line through exactly 2 of them.”

But this was Sylvester’s conjecture, which since then became a theorem, so we are done.

(End of Proof.)

Austin, 13 January 1996

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

transcribed by Corrado Cantelmi

revised
18-Mar-2012
