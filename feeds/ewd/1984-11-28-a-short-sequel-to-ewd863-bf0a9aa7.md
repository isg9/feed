---
title: "A short sequel to EWD863 (EWD903)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD903.html
published: "1984-11-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD903.PDF
---

# A short sequel to EWD863

A short sequel to EWD 863
In EWD863, I left at the end the derivation of de Morgan's Laws as an exercise to the reader. The following proof, however, is too beautiful to remain unrecorded. I recall —with numbers as in EWD863— the axioms

[ P ∧ Q  ≡  P  ≡  Q  ≡  P ∨ Q ]  (9)

[ P ∨ ¬Q  ≡  P ∨ Q  ≡  P ](14)

and the theorems

[ ¬¬Q ≡ Q ] (19)

[¬(P ≡ Q) ≡ ¬P ≡ Q ] . (20)

From (14), we derive with P := ¬P and P,Q := Q,P respectively:

[ ¬P ∨ ¬Q  ≡  ¬P ∨ Q  ≡  ¬P ]

[ Q ∨ ¬P  ≡  Q ∨ P  ≡  Q ] ;

from those two with Leibniz's Principle (and symmetry of v and ≡ )

[ ¬P ∨ ¬Q  ≡  ¬P  ≡  Q  ≡  P ∨ Q ] ;

from that one with(20)

[ ¬P ∨ ¬Q  ≡  ¬( P  ≡  Q  ≡  P ∨ Q ) ] ,

and finally with (9)

[ ¬P ∨ ¬Q  ≡  ¬( P ∧ Q ) ] . (25)

With (19) —and [¬P ≡ ¬P], which is a syntactic descendant
of [ P ≡ P ]—

[ ¬P ∧ ¬Q  ≡  ¬( P ∨ Q ) ] (26)

follows readily from (25).

Austin, 2 Dec. 1984

prof.dr.Edsger W.Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188
United States of America

PS. The single substitution P,Q := ¬Q,P into (14), yielding

[ ¬Q ∨ ¬P  ≡  ¬Q ∨ P  ≡  ¬Q ]

would have sufficed in subsequent combination with (14).
(End of PS.) EWD.

transcribed by Germán González-Morris
revised Fri, 17 Dec 2010
