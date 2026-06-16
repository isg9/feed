---
title: "For the record: ETAC and the couples (EWD1103)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1103.html
published: "1991-06-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1103.PDF
---

# For the record: ETAC and the couples

ETAC learned the following problem from JLA van de Snepscheut (who probably heard it from J Misra). Of a finite number of couples we are told the following two things:

(i)
the oldest man is as old as the oldest woman

(ii)
if two couples swap wives, the youngest of the one new couple is as young as the youngest of the other new couple.

Show that in each couple, man and woman have the same age.

The ETAC came with the following solution. Let dummies x, y range over the couples; let the ages of man and woman of couple x be denoted by m.x and f.x respectively. Our proof obligation is then

(0)     (Ax :: m.x = f.x)

while we have been given

(1)     (↑y :: m.y) = (↑y :: f.y)            and

(2)     (Ax, y :: m.x↓f.y = f.x↓m.y)

Note that (ii) states (2) with the range x≠y, but that the range can be extended to include x=y.

We observe for any x

m.x  = f.x

=        { ↑↓ calculus: p ≤ p↑q and a ≤ b ≡ a = a↓b}

m.x↓(↑y :: m.y) = f.x↓(↑y :: f.y)

=        {(1)}

m.x↓(↑y :: f.y) = f.x↓(↑y :: m.y)

=        {↑↓ calculus: ↓ distributes over ↑}

(↑y :: m.x↓f.y) = (↑y :: f.x↓m.y)

=        {(2)}

true   ,

and thus (0) has been proved.

The first step is not surprising: in order to use (1) and (2), ↑ and ↓ have to be introduced into the calculation.

Nuenen, 29 June 1991

prof.dr. Edsger W.Dijkstra,

Department of Computer Sciences,

The University of Texas at Austin,

Austin, TX 78712–1188

USA

Transcribed by Guy Haworth

Revised Sun, 6 Jan 2008.
