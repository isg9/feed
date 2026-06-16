---
title: "An intriguing example (EWD758)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/ewd758.html
published: "1980-11-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD758.PDF
---

# An intriguing example

In the following all variables and all elements of the infinite arrays f [0...] and g[0...] are of type natural number.
Array f is ascending, i.e.

(A x : x≥0 : f [x] ≤ f [x+1]) (0)

and unbounded, i.e.

(A y : y≥0 : (E x : x≥0 : f [x] > y)) (1)

As a result of (1)

prog 0:do f [x] ≤ y → x := x+1 od

terminates. Also — obviously —

prog 1:do f [x] > y → g[y]:= x; y := y+1 od

terminates. The “combined” program

x, y := 0,0;

do f [x] ≤ y → x := x + 1;

⫿ f [x] > y → g[x] := x; y := y+1

od

obviously fails to terminate. Hence, x and y are both unbounded: more and more of f will be taken into account, and more and more of g will be defined.
From 0 we derive

(N i : i≥0 : f [i] ≤ f [x]) ≥ x+1 (2)

The weakest precondition that x := x+1 establishes

(N i : i≥0 : f [i] ≤ y) ≥ x(3)

is, according to the axiom of assignment,

(N i : i≥0 : f [i] ≤ y) ≥ x+1 ,

which, on account of (2), is implied by f [x] ≤ y; hence, the first alternative leaves (3), which is established by x, y := 0,0, invariant.
So does the second alternative (obviously).

From f [x] > y we derive, on account of (0)

(N i : i≥0 : f [i] ≤ y) ≤ x ,

which, in conjunction with (3) allows us to conclude that, then, (N i : i≥0: f [i] ≤ y) = x. Hence, we have the second invariant

(A j : 0≤j<y : g[j] = (N i : i≥0 : f [i] ≤ j)) (4)

and this is exactly the property I wanted to prove about my program
*              *
*

The example is — see EWD753— inspired by the theorem of Lambek and Moser, a theorem Wim Feijen found when looking for functions to be programmed in SASL.
As a matter of fact, my “combined” program was not the first program I wrote to solve this problem: it is a direct translation of the
following SASL definitions I wrote first: (my syntax)

def k x y (p:q) = (5)

if p ≤ y → k (x+1) y q

⫿ p > y → x : k x (y+1) (p:q)

fi

def g = k 0 0 f

But even the proof of the fact that g is ascending
—which in the iterative program follows trivially from the equally obvious invariant

y = 0 cor g[y−1] ≤ x —

was very painful when I tried a proof technique � la EWD749 which does justice to the “functional” nature of applicative languages:
(5) is expressed in terms of tails, my proof is in terms of finite prefixes. I think I should ask an expert (See EWD759.)

Plantaanstraat 59 November 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
