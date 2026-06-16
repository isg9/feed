---
title: "Some useful formulae (with A.J.M. van Gasteren) (EWD877)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD877.html
published: "1984-02-08T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD877.PDF
---

# Some useful formulae (with A.J.M. van Gasteren)

AvG34 / EWD877

Some useful formulae

Let f be a propositional expression and R a propositional variable. Using the slash "/" to denote "substituted for", we have the general equivalences

f ≡ ( f ( false / R ) ∨ R ) ∧ ( f ( true / R ) ∨ ¬ R )

f ≡ ( f ( true / R ) ∧ R ) ∨ ( f ( false / R ) ∧ ¬ R ) .

(We don't prove this; it should probably be proved using induction over the syntax.)

The purpose of this note is to draw the attention to a few special instances.

P ≡ Q ∨ R ≡ (( P ≡ Q ) ∨ R ) ∧ ( P ∨ ¬ R )

P ≡ Q ∧ R ≡ (¬ P ∨ R ) ∧ (( P ≡ Q ) ∨ ¬ R ) .

(Substituting in the latter formula ¬P for P we get

P ≢ Q ∧ R ≡ ( P ∨ R ) ∧ (( P ≢ Q ) ∨ ¬ R ) .

We used the last formula recently to prove

( y ≢ z ∧ a ) ∧ ( z ≢ y ∧ b ) ≡

( y ∨ a ) ∧ ( z ∨ b ) ∧ (( y ≢ z ) ∨ ¬ ( a ∨ b )) .)

Plataanstraat 5
5671 AL NUENEN
The Netherlands 8 February 1984
drs. A.J.M. van Gasteren
prof.dr. Edsger W. Dijkstra

transcribed by Alejandro Ruiz
revised Wed, 12 Nov 2008
