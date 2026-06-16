---
title: "Simplifying a proof in our book (EWD1109)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1109.html
published: "1991-09-25T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1109.PDF
---

# Simplifying a proof in our book

In [0], pp.66-69, we show how the conditional distribution of  ∧  over  ∀  can be derived from the one-point rule and the other axioms. I dragged sets into the picture, for which misbehaviour I apologize; here is a simpler argument. Representing “the non-empty range” for the dummy  x  by r.x  ∨  [x=y] we show

[〈∀x : r.x ∨ [x=y]: t.x ∧ Q〉 ≡ 〈∀x : r.x ∨ [x=y]: t.x〉∧ Q ]

To this end we observe

〈∀x: r.x ∨ [x=y]: t.x ∧ Q〉

=         {splitting the term}

〈∀x: r.x ∨ [x=y]: t.x〉 ∧ 〈∀x: r.x ∨ [x=y]: Q〉

=         {see (*) below}

〈∀x: r.x ∨ [x=y]: t.x〉 ∧ Q

(*) We observe

〈∀x: r.x ∨ [x=y]: Q〉

=         {splitting the range}

〈∀x: r.x: Q〉∧〈∀x: [x=y]: Q〉

=         {one-point rule}

〈∀x: r.x: Q〉∧ Q

=         {see (**) below)

Q

(**) We observe

[Q ⇒〈∀x:r.x: Q〉]

=         { ⇒ distributes —like ∨— over ∀ in consequent}

[〈∀x: r.x: Q ⇒ Q〉]

=         {pred. calc}

[〈∀x: r.x: true〉]

=         {pred. calc., e.g. [0], p.66, (90)]

[true]

=         {pred. calc.}

true

[0] Edsger W. Dijkstra & Carel S. Scholten Predicate Calculus and Program Semantics, Springer-Verlag New York Inc., 1990

Austin, 25 September 1991

prof.dr. Edsger W.Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712–1188
USA

transcribed by Guy Haworth
revised Sat, 13 Dec 2008
