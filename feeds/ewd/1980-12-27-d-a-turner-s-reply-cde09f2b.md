---
title: "D.A.Turner’s reply (EWD770)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/ewd770.html
published: "1980-12-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD770.PDF
---

# D.A.Turner’s reply

D.A. Turner's reply.
In reply to EWD759, D.A. Turner sent me the following proof by returning mail.
“Thank you for your “somewhat open letter”, which arrived yesterday. You pose several task —the order in which I
have decided to tackle them is to establish first the precise formal relationship between f and g.
First some notation. I shall call the elements of a list f ,   f0 f1 f2 etc. and I shall use the notation

[h(i)]Bi=A

for the list [h(A), h(A+1), ..., h(B)].
Now I define a function “upto”

upto i f = least j ≥ 0 such that fj > i(upto.0)

The claim to be established is that

g = [upto i f ] ∞ i=0(Theorem 1)

From upto.0 we deduce the following two propositions, which could be considered a SASL definition of upto

a > i ⊢ upto i (a:f ) = 0 (upto.1)

a ≤ i ⊢ upto i (a:f ) = 1 + upto i f(upto.2)

We have also your definition of the function “k”

p ≤ y ⊢ k x y (p:q) = k (x+1) y q(k.1)

p > y ⊢ k x y (p:q) = x : k x (y+1)(p:q) (k.2)

From these four propositions we shall deduce the following generalization of Theorem 1.
Theorem 0. k x y f = [x + upto i f ] ∞ i=y
Proof by structural induction on f, which is an infinite list of integers.
case ΩL (Note — we need to distinguish between ΩL, the undefined element in the space
to which f belongs, and ΩI, the undefined integer. The relationship between them
is ΩL = [ΩI] ∞ i=0     )

k x y ΩL = ΩL from k.1, k.2 by case exhaustion

whereas

[x + upto i ΩL] ∞ i=y

= [x + ΩI] ∞ i=y     from upto.1, upto.2

= [ΩI] ∞ i=y        properties of Ω

= ΩL as required

case p:f

k x y ( p:f )

= [x]p−y ++ k x p ( p:f ) by repeated appl. of k.2

= [x]p−y ++ k (x+1) p f ) by k.1

= [x]p−y ++ [(x+1) + upto i f ] ∞ i=p ex hyp.

= [x]p−y ++ [x + (1 + upto i f )] ∞ i=p properties of +

= [x]p−y ++ [x + upto i (p:f ))] ∞ i=p by upto.2

= [x + upto i (p:f ))] ∞ i=y by upto.1 and rearranging

QED Theorem 0.
Whence, since g = k 0 0 f, we have immediatly
Theorem 1 g = [upto i f ] ∞ i=0
Also you asked me to establish that g is
A) ascending and B) unbounded, given appropriate assumptions about f.
This now follows easily from the above. (Relaxing the level of formality somewhat) we have:
A) from upto.0 it follows (by transitivity of “>” that “upto i f " is an ascending function of i. Therefore, whatever
the nature of f, g is ascending.
B) Let us define “f is unbounded” to mean, “for any N≥0, there is a j ≥0 such that fj > N. Assume f is
unbounded (f ascending not relevant. Then, from upto.0,

upto i f is defined for all i ≥0

Given any N≥0, define j = max{f0, ..., fN} then upto j f = gj exists, and by construction > N. So g too is unbounded.
*              *
*

So far Turner's reply. I like Turner’s proof, and in view of the fact that Turner answered me by returning mail it
would be misplaced to complain too much about the fact that, in “case p:f ” the definitely less interesting case p<y hasn’t been dealt with explicitly.
I am slightly uneasy in “case ΩL” —particularly after the parenthetical remark explaining
the difference between ΩL and ΩI— about
the justification “from k.1, k.2 by case exhaustion”. My uneasyness is certainly caused by lack of familiarity how to deal with Ω. Take

funny (p:q) =
if p≥10 → 1: funny q

⫿ p<10 → 1: funny q

fi

Is funny ΩL = ΩL? Or is funny ΩL = ones
(with ones = 1: ones)? I expect the first answer though I would prefer the second one, if
I am giving full weight to the remark in [1], page 37).

The first point to be made is that in reasoning about SASL programs, Ω can be treated just like any other value
as regards being substitutable in equations

Perhaps I have failed to fathom the complete depth of the constrained “as regards being substitutable in equations”.
The correspondence was triggered by remarks in [1] such as the recommendation of applicative programming (pg 14):
The proofs (like
the programs themselves) are very much shorter than the proofs of the corresponding imperative programs.

I had my doubts,
which have not been dispelled by the comparison of Turner’s proof with the one given in EWD758.
[1] Turner, D.A. “Program
Proving and Applicative Languages”, August 1980

Plantaanstraat 527 December 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
