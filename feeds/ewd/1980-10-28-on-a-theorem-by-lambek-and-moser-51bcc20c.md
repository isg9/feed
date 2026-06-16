---
title: "On a theorem by Lambek and Moser (EWD753)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD753.html
published: "1980-10-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD753.PDF
---

# On a theorem by Lambek and Moser

While looking for interesting functions mapping
sequences of integers into sequences of integers,
W.H.J. Feijen found in [1] the following results
attributed to J. Lambek and L. Moser.
Let f be an ascending continued concatenation
of natural numbers, more precisely:

(A x : x ≥ 0 : 0 ≤ f x ≤ f (x+1)) ,

such that, furthermore, f x exceeds any given
bound F for sufficiently large x, more precisely:

(A F : F ≥ 0 : (E X : X ≥ 0 : (A x : x ≥ X : f x > F))) .

Let the function C with g = C f be defined by the relation for all y ≥ 0

g y = (N x : x ≥ 0 : f x ≤ y)

(The right-hand side being read as “the number of distinct natural values x such that
f x is at most y”).
Obviously, g is also an ascending continued concatenation such that g y exceeds any given bound G for sufficiently large y.
The first result attributed to Lambek and Moser is that then f = C g, as is illustrated by the example

0123456789......

f :011366781010......

g :1334446788......

Let for any continued concatenation q of natural numbers the function D be defined by the relation for all n ≥ 0

D q n = n + (q n) .

In the above example we would have

0123456789......

D f :0236101113151819......

D g :14578912141617......

The second result attributed to Lambek and Moser is that D f and D g form a partitioning of the natural numbers.
For people willing to generalize pictures the visualization “proves” these results. The elements of f and D f have been
represented by vertical bars, one wide and of the corresponding length, those of g and D g have been represented by a horizontal
ones. The mapping C then becomes a reflection.

Tracing the plot, we can place the bars in the order of increasing length: bar 14 is next (horizontally), bar 15 vertically, bars 16 and 17 again horizontally, etc.

[1]
Honsberger, Ross

“Ingenuity in Mathematics”

New Mathematical Library, Vol 23

Mathematical Association of America (1970)

Plantaanstraat 59 October 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
