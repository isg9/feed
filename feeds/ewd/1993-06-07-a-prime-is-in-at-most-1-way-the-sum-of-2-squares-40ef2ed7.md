---
title: "A prime is in at most 1 way the sum of 2 squares (EWD1155)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1155.html
published: "1993-06-07T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1155.PDF
---

# A prime is in at most 1 way the sum of 2 squares

EWD 1155

A prime is in at most 1 way the sum of 2 squares

EWD1154 dealt with D.Zagier’s proof that a prime of the form 4k+1 is the sum of 2 squares. In fact, such a prime is in only 1 way the sum of 2 squares. In this note we show this by proving that if an odd n is the sum of 2 different pairs of squares, then that n is not prime.

Let an odd n be the sum of 2 squares; then the one square is odd, the other is even: the squares are of different parity. Let n be the sum of 2 squares in 2 ways; then there exist positive a, b, c, d such that

(0)
(a + b)� + (c – d)� = n

(a – b)� + (c + d)� = n

(Here a is the average of the numbers of the one parity, c is the average of those of the other parity. Because we are considering distinct square decompositions, also b and d can be chosen positive.)

Eliminating n from (0) by equating the left-hand sides, we deduce after simplification

(1)
ab = cd       ,

from which we deduce the existence of positive r, s, t, v such that

(2)
a = sv

b = rt

c = st

d = rv

(Consider
“s := a gcd c;
v := a/s;
t := c/s;
r := b/t”.)

Now we observe

n

=         {(0)}

(a + b)� + (c – d)�

=         {(1)}

a� + b� + c� + d�

=         {(2)}

s�v� + r�t� + s�t� + r�v�

=         {algebra}

(s� + r�) ∙ (t� + v�)

and because the 4 variables are positive, the two factors are each at least 2, and hence n is not a prime number.

*         *         *

The above was written down in Abilene State Park. In contrast to the proof discussed in EWD1154, I designed this proof myself, but the title of this note does not mention “derivation” of the proof, since I did not “derive” it in any technical sense.

I have considered investigation of the situation x�+y� = n ∧ u�+v� = n ∧ prime.n with the aim of showing (x,y) = (u,v) ∨ (x,y) = (v,u), but rejected that approach for the disjunction, and for the fact that I saw no way of using n’s primality. So I did some shunting and set myself to show that n was composite by writing it as a product of 2 plurals. I knew my complex numbers, in particular, that the modulus of a product is the product of the moduli, and then discovered that there was no point in looking at (x+yi)�(u+vi). Hence

(3)
(sv – rt)� +
(st + rv)� =
(s� + r�) �
(t� + v�)

—the 2 expressions for the modulus of (s + ri) � (t + vi), which do equate a sum of squares to a product— has to be used differently. The right-hand side being even in r, it also equals (sv + rt)� + (st – rv)�, and now we see the a, b, c, d entering the picture. The introduction of a � b and c � d circumvented the disjunctive complication of comparing unordered pairs.

I think I knew (3) outside the context of complex numbers as well; it is very common to separate in (a � b)� the squares from the cross product, as in

(a + b)� =
(a – b)� + 4ab

(a + b)� +
(a – b)� = 2(a� + b�)       .

The proof reported provides a striking example of a proof in which the algebra is totally trivial while all subtlety has been invested in the decision what to name.

Austin, 7 June 1993

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
24-Dec-2011
