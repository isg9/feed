---
title: "Potter’s proof of disjunction’s symmetry (EWD1096)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1096.html
published: "1991-03-04T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1096.PDF
---

# Potter’s proof of disjunction’s symmetry

EWD 1096

Potter’s proof of disjunction’s symmetry

Dr. Walter M. Potter of Southwestern University, Georgetown, TX 78626, taught me that the symmetry of the disjunction can be derived from my other axioms:

P ≡ Q

=         {idempotence of ∨}

(P ≡ Q) ∨ (P ≡ Q)

=         {∨ distributes forwards over ≡}

(P ≡ Q) ∨ P ≡ (P ≡ Q) ∨ Q

=         {∨ distributes backwards over ≡, twice}

P ∨ P ≡ Q ∨ P ≡ P ∨ Q ≡ Q ∨ Q

=         {idempotence of ∨, twice}

P ≡ Q ∨ P ≡ P ∨ Q ≡ Q       ;

From the first and the last line (and the properties of ≡) follows

[Q ∨ P ≡ P ∨ Q]      .

Q.E.D.

Austin, 4 March 1991

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
24-Dec-2011
