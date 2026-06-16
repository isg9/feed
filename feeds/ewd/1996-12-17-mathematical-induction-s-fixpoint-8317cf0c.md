---
title: "Mathematical induction’s fixpoint (EWD1253)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1253.html
published: "1996-12-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1253.PDF
---

# Mathematical induction’s fixpoint

Mathematical induction's fixpoint

Let r be a left-founded relation, i.e. let

(0)          〈 ∀x :: [x ⇒ r ; x] ⇒ [ x ⇒ false ]  〉     ;

an appeal to what is called "mathematical induction" means that a proof obligation of the form

(1)          [ x ⇒ false ]

is replaced by the formally weaker obligation

(2)          [x ⇒ r ; x]      or      [ x ∧ ¬ ( r ; x ) ⇒ false ] .

Proving (2) by mathematical induction leads to the (subsequently simplified) obligation

(3)          [ x ∧ ¬ ( r ; x ) ⇒ r; ( x ∧ ¬ ( r ; x ))]
≡               {shunting}
[x ⇒ r ; x ∨ r ; ( x ∧ ¬ ( r ; x ))]
≡               { ; monotonic in 2nd argument}
[x ⇒ r ; x]

i.e. (2)! The decision to prove by mathematical induction is idempotent. With thanks to the ETAC.

Nuenen, 17 December 1996

prof.dr. Edsger W.Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188, USA

transcribed by Jose Alejandro Ruiz Delgado
revisedSun, 31 Aug 2008
