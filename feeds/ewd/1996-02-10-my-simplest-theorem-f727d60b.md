---
title: "My simplest theorem (EWD1232)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1232.html
published: "1996-02-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1232.PDF
---

# My simplest theorem

Theorem Any natural number that has a divisor greater than itself equals zero.

Proof We observe for any natural n, d, q

n = d∙q  ∧  d > n

=        { Leibniz }

n = d∙q  ∧  d > d∙q

=        { d > 0 }

n = d∙q  ∧  1 > q

=        { q is natural }

n = d∙q  ∧  q = 0

⇒        { Leibniz }

n = 0

( End of Proof )

At least twice —EWD1088 & EWD1170— I had used that 0 is the only natural number with infinitely many divisors —e.g. 2k for any k— but I never took the trouble to prove it, and that probably explains why I missed the above.

Austin, 10 February 1996

prof. dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Tristam Brelstaff

modified Mon, 10 Dec 2007.
