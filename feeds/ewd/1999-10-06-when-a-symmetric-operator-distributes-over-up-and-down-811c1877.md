---
title: "When a symmetric operator distributes over (up) and (down) (EWD1292)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1292.html
published: "1999-10-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1292.PDF
---

# When a symmetric operator distributes over (up) and (down)

When a symmetric operator distributes over ↑ and ↓

To begin with we postulate (0) and (1):

(0)    relation ⊑ is reflexive and antisymmetric,

i.e., for all x, y

x=y ≡ x⊑y ∧ y⊑x

(1)    for any x,y there exists a (unique) value, which we denote by x↑y, that satisfies for all z

x↑y ⊑ z ≡ x⊑z ∧ y⊑z .

Remark Since the uniqueness can be proved, it need not be postulated; hence the parentheses.
(End of Remark.)

It then follows that (for all x,y)

(2)    ↑ is idempotent, symmetric, and associative

(3)    x ⊑ x↑y

(4)    x = x↑y ≡ y⊑x

(5)    ⊑ is transitive (hence —see (0)— ⊑ is what is known as “a partial order”)

(6)    ↑ is monotonic

We'll just prove (6).

Proof We calculate for any triple x,y,z :

x↑z ⊑ y↑z

≡ {(4)}

y↑z = (y↑z)↑(x↑z)

≡ {(2)}

y↑z = (y↑x)↑z

⇐ {Leibniz’s Principle}

y = y↑x

≡ {(4)}

x ⊑ y

To the above we add “the dual” (0′) through (6′) obtained from (0) through (6) by

(α) interchanging the arguments of ⊑ , and

(β) replacing ↑ by ↓ and vice versa.

Remark The reader may note that, thanks to the symmetry of ∧ , (0′) is equivalent to (0).
The “and vice versa” has been added so as to make (β) —like (α)— its own inverse.
(End of Remark.)

And this concludes our summary of Lattice Theory.

*               *

*

Now we are ready to formulate and prove the following simple theorem.

Theorem Let the symmetric infix operator ● distribute over ↑ and ↓ Then

(7)    (x↑y)●(x↓y) = x●y

Proof To begin with we observe for any x,y

(x↑y)●(x↓y)

= {● distributes over ↑}

(x●(x↓y))↑(y●(x↓y))

= {● distributes over ↓, twice}

(x●x ↓ x●y)↑(y●x ↓ y●y)

⊑ {(3') and (6), twice}

x●y ↑ y●x

= {symmetry of ● , idempotence of ↑}

x●y ,

i.e.,

(8)    (x↑y)●(x↓y) ⊑ x●y .
.

By dualization we conclude from (8):

(8′)    x●y ⊑ (x↓y)●((x↑y) ,

and thanks to the symmetry of ● and (0), the conjunction of (8) and (8′) yields (7).
(End of Proof).

*               *

*

The above theorem and proof emerged yesterday afternoon when K. Rustan M. Leino joined the session
of the ATAC and wanted to show us some aspects of his proof of the fact that the product of two
natural numbers equals that of their GCD and their LCM (what he did). The decision to go from there
to lattices in general was a minor step. Acknowledgements are further due to Dr Rajeev Joshi, who
contributed to what we presented here.

Austin, 6 October 1999

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Hasnain Mujtaba

revised
08-Jan-2015
