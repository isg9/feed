---
title: "A theorem about “factors” perhaps worth recording (EWD1207)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1207.html
published: "1995-05-15T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1207.PDF
---

# A theorem about “factors” perhaps worth recording

A theorem about "factors" perhaps worth recording

Let \ be defined by

(0)    [x;y ⇒ z]  ≡  [y ⇒ x\z]

(0) defines x\z as the weakest solution of

y:  [x;y ⇒ z]

and yields with instantiations  y := x\z  and  z := x;y  respectively

(1)    [x; x\z  ⇒  z]

(2)    [y ⇒ x\(x;y)]

(We have given \ a higher binding power than ; .)

About the transpose ~ (prefix) —which others call the converse º (postfix)— I shall use the Dedekind Law

(3)   [x;y ∧ z  ⇒  x; (y ∧ ~x; z)]  .

*                 *

*

We shall now prove

(4)    [p;q ∧ ~p\r  ⇒  p;r ]

To this end we observe for any  p, q, r

p;q ∧ ~p\r

⇒       { (3) with x, y, z := p, q, ~p\r }

p ; (q ∧ ~p ; ~p\r )

⇒       {monotonicities, (1) with x, z := ~p, r }

p;r

I used (4) to prove the theorem of section 2.3 of "A Graphical Calculus" by Sharon Curtis and Gavin Lowe, Oxford University Computing Laboratory, Parks Road, Oxford, OX1 3QD; the formulation of (4) was triggered by their note.

An alternative formulation of (4) that incorporates the antimonotonicity of \ in its left argument is

(5)    [~p ⇒ s]  ⇒  [p;q ∧ s\r  ⇒  p;r]

I think the theorem is worth recording though not worth remembering.

Austin, 15 May 1995

prof.dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 - 1188 USA

Transcribed by Guy Haworth.

Lastr revised Sun, 30 Dec 2007.
