---
title: "Fibonacci numbers and Leonardo numbers (EWD797)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD797.html
published: "1981-07-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD797.PDF
---

# Fibonacci numbers and Leonardo numbers

EWD 797

Fibonacci numbers and Leonardo numbers.

(The following formal derivations and computations
are absolutely elementary and without scientific in-
terest. But I am interested in some numbers and
need the formulae, and learned that working on a
scratch pad I make too many mistakes. Hence.)

The Fibonacci numbers are given by

F0 = 1     F1 = 1     Fn+2 = Fn+1 + Fn (or : Fn+2 – Fn+1 – Fn = 0).

The analytical solution of a homogeneous recurrence
relation like this is found by solving first the cor-
responding characteristic equation which one gets
by "trying" an F of the form Fn = xn :

(0) x2 – x – 1 = 0

This equation has two different roots, which I
shall denote by α and β

(1) α = ½ + ½√5 = 1.618034        10log α = 0.208988

β = ½ – ½√5 = – .618034

Obvious properties are

(2) α + β = 1          α∙β = –1

α2 = α + 1 , α3 = 2∙α + 1 , α4 = 3∙α + 2 , etc.

Because α ≠ β , αn and βn give rise to linearly
independent sequences and each solution of the
homogeneous recurrence relation is of the form

(3) Fn = X∙αn + Y∙βn

where the constants X and Y are determined by
solving the set of linear equations

(4)
X + Y = F0

α∙X + β∙Y = F1

Multiplying the second equation by α we get —see (2)—

(α + 1)∙X – Y = α∙F1 , and hence
(α + 2)∙X = F0 + α∙F1 , hence

F0 + (½ + ½ √5)∙F1    2∙F0 + (1 + √5)∙F1 (5 – √5)
X = ━━━━━━━━━━━━ = ━━━━━━━━━━━━ ∙ ━━━━━
2 ½ + ½ √5 (5 + √5) (5 – √5)

1
= ━━ (10∙F0 – 2∙F0∙√5 + 4∙F1∙√5) =
20

5∙F0 + (2∙F1 – F0)∙√5
= ━━━━━━━━━━━━ (5)
10

and, for reasons of symmetry

(5∙F0 – (2∙F1 – F0)∙√5)
Y = ━━━━━━━━━━━━ (5')
10

For the Fibonacci numbers we substitute F0 = F1 = 1
and find, according to (3)

(6) Fn = (½ + ⅟10√5)∙αn + (½ – ⅟10√5)∙βn

= .723607∙αn + .276393∙βn

* *
*

We now switch to the Leonardo numbers given
by L0 = 1 L1 = 1 Ln+2 = Ln+1 + Ln + 1 . This
recurrence relation is not homogeneous but
—because x = 1 is not a root of (0)— this is only
an apparent complication: (Ln+2 + 1) = (Ln+1+1) + (Ln + 1),
and we immediately derive

(7) Ln = 2∙Fn – 1 .

The nth Leonardo tree has Ln vertices. The
(n+2)th Leonardo tree is a binary tree, of which
the (n+1)th and the nth Leonardo tree are
the two subtrees. A number that is perhaps of some
interest is the distance from the root summed
over the vertices of the nth Leonardo tree.
Denoting this quantity by Kn we derive from
the definition

(8) Kn+2 = (Kn+1 + Ln+1) + (Kn + Ln)
or
(Kn+2–2) = (Kn+1–2) + (Kn–2) + (Ln+1+1) + (Ln+1)

or with

(9) Kn = 2∙(Hn + 1) or Hn = (Kn–2)/2
(10) Hn+2 = Hn+1 + Hn + Fn+2

Solving (10) for Fn+2 and taking the recurrence
relation for the F's into account one finds that the
H's satisfy a homogeneous linear recurrence relation
with (x2 – x – 1)2 = 0 as characteristic equation. (This
is a special case of a more general theorem of
which I was not aware.) Hence the general form
of Hn is

(11) Hn = (a + n∙A)∙αn + (b + n∙B)∙βn

where the constants a, A, b, and B are determined
by solving —see (2)—

(12)

a +

b

= H0

α∙a +

α∙A +

β∙b +

β∙B = H1

(α + 1)∙a +

(2∙α + 2)∙A +

(β + 1)∙b +

(2∙β + 2)∙B = H2

(2∙α + 1)∙a +

(6∙α + 3)∙A +

(2∙β + 1)∙b +

(6∙β + 3)∙B = H3

We eliminate a and b with

(13) y0 = H2 – H1 – H0, y1 = H3 – H2 – H1

(α + 2)∙A   +
(β + 2)∙B  =  y0

(3∙α + 1)∙A   +

(3∙β + 1)∙B  =  y1

,

which leads to

5∙A  +
5∙B  =  z0

α∙(5∙A)  +
β∙(5∙B)  =  z1
with

(14) z0 = 3∙y0 – y1 , z1 = 2∙y1 – y0

This set of equations is of the same form as (4).
Hence we have

5∙z0 + (2∙z1 – z0)∙√5
(15) A = ━━━━━━━━━━━
50

5∙z0 – (2∙z1 – z0)∙√5
(15') B = ━━━━━━━━━━━
50

We can eliminate A and B from (12) with

y3 = –2∙H3 + 3∙H2 + 6∙H1 – H0

5∙a    +
5∙b
= 5∙H0

α∙(5∙a) +
β∙(5∙b)
= y3

which is again of the form (4). Eliminating y3, we get

25∙H0 + (–4∙H3 + 6∙H2 + 12∙H1 – 7∙H0)∙√5
(16) a = ━━━━━━━━━━━━━━━━━━━━━━━
50

25∙H0 – (–4∙H3 + 6∙H2 + 12∙H1 – 7∙H0)∙√5
(16') b = ━━━━━━━━━━━━━━━━━━━━━━━
50

Let us apply these formulae with the numerical
values of K0, K1, K2, K3 and then check them against
the numerical value of K4 . (It is a long time ago
since I made my last check.)

n:

Ln:

Kn:

Hn

From (13) : y0 = 2      y1 = 3

0

1

0

–1

From (14) : z0 = 3      z1 = 4

1

1

0

–1

From (15) :

2

3

2

0

A =

15 + 5∙√5

━━━━━━

50

=

3 + √5

━━━━

10

3

5

6

2

4

9

16

7

B =

3 – √5

━━━━

10

From (16)
a =
–25 + (–8 – 12 + 7)∙√5

━━━━━━━━━━━━━━

50
=
–25 –13∙√5

━━━━━━━━

50

b =
–25 + 13∙√5

━━━━━━━━

50
.

In order to check these values for n = 4 we compute

(a + 4∙A)∙α4 = ⅟50∙(–25 – 13∙√5 + 60 + 20∙√5)∙(3∙α + 2)
= ⅟100∙(35 + 7∙√5) (7 + 3∙√5) = ⅟100∙(245 + 105 + 154∙√5) =
3½ + 1.54∙√5 . This is OK and that is very encouraging.

We are now in a position to compute the asymp-
totic behaviour of the average distance from the root

Kn

━━

Ln

=

2∙Hn + 2

━━━━━

2∙Fn – 1

→

Hn

━━

Fn

→

⅟10∙(3 + √5)

━━━━━━━━

⅟10∙(5 + √5)

=

5 + √5

━━━━

10
∙

With N the number of points, the average distance
grows as ⅟10∙(5 + √5)∙αlog N = .723607∙αlog N , a growth
rate I would like to compare to the one of the completely
balanced binary tree. The number of nodes in the
nth binary tree equals 2n+1 – 1 . The sum over
its nodes of their distances from the root is
(S i : 0 ≤ i ≤ n : i∙2i) = (n–1)∙2n+1 + 2 . With N
the number of points, the dominant term of the
growth rate of the average distance from the
root is therefore 2log N . For the Leonardo
trees it is .723607∙αlog N = 1.042296∙2log N .
The ratio is —as was to be expected— larger than
1, but only very little so. (I am not convinced
of the relevance of the notion "average distance
from the root" ; it has the advantage that the
above estimations can be derived by elementary
means.)

* *
*

We know that, with a given number, taking
away the largest possible Leonardo number
and repeating this process on the remainder,
we decompose the given number, x say, in the
minimum number f(x) of Leonardo numbers.
What is the average value of f(x) when x
ranges over the first N natural numbers ?

Defining Di = (Sx : 0 ≤ x ≤ Li+1 : f(x)) , we have
D0 = 0 , D1 = 3 , Dn+2 = Dn+1 + Dn + Ln+1 + 2 ,
hence D2 = 6 , D3 = 14 , D4 = 27 , etc.

This is the moment I am going to reap the fruit
of (13), (14), and (15). With Hn = Dn + 1 we have
Hn+2 = Hn+1 + Hn + 2∙Fn+1 , and (11) is applicable.
We have H0 = 1 , H1 = 4 , H2 = 7 , and H3 = 15. From
(13) y0 = 2 , y1 = 1 ; from (14) z0 = 2 , z1 = 6 , and
from (15) A = ⅟50∙(10 + 10∙√5) = ⅟5∙(1 + √5) . Hence the
dominant term of Hn (and Dn) is ⅟5∙(1 + √5)∙n∙αn .
The leading term of Ln+1 (=2∙Fn+1 – 1) is ⅟5∙(5 + √5)∙αn+1 =
⅟10∙(1 + √5) (5 + √5)∙αn .

The growth rate of the average value of f(x) is
that of Hn/Ln+1 , i.e. 2∙n/(5 + √5) = ⅟10∙(5 – √5)∙n =
.276393∙αlog N .

Analogously to the perfectly balanced binary trees
we can replace the Leonardo numbers by Bn = 2n+1 – 1.
Let f'(x) be the minimum number of B's with sum x
and let Cn = (S : 0 ≤ x ≤ Bn : f'(x)). We find
Cn = (n + 1)∙2n . In this case the growth rate of the
average value of f'(x) is —not surprisingly—
Cn/Bn = ½ (n + 1) = ½∙2log N = 4log N . Comparing
this with the case of the Leonardo numbers

.276393 αlog N = .796243 ∙ 4log N

and this time the ratio is markedly smaller than 1.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

12th July 1981

prof.dr. Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Sam Samshuijzen

revised Tue, 7 Jun 2005
