---
title: "A not so simple theorem about undirected graphs (EWD646)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD646.html
published: "1977-10-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD646.PDF
---

# A not so simple theorem about undirected graphs

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

A not so simple theorem about undirected graphs. See pg.2.
(The other day Alain Martin told me that C.S.Scholten had shown him a
rather complicated proof for what seemed a fairly simple theorem, and
commucated the theorem to me. I felt challenged and tried to find a “simple” proof
myself. The proof that I found is recorded below, partly because I think my
proof simple enough to have some beauty, partly because perhaps the theorem
is not so simple after all: it took me a full day to prove it and another
five hours to write the following in manuscript.)
In a finite, undirected graph we call two nodes that are directly
connected by an edge of the graph each other’s “neighbours”. Let P and Q be
two different nodes of the graph. We define a “path from P to Q ” as a
sequence of nodes, starting with P and ending with Q , such that no node
occurs more than once in it and any two adjacent nodes in the sequence are
each other’s neighbours in the graph. Of such a path, P and Q are called
“the terminal nodes”, the ones in between are called “the internal nodes”.
(If P and Q are not each other’s neighbours, a path from P to Q has
at least one internal node.)
The theorem states that for any pair of different nodes A and B that
are not each other’s neighbours, there exists a node C that is an internal
node of any path from A to B , or there exists a pair of paths from A to
B that have no internal node in common.
In order to prove it, we arbitrarily select one path from A to B and
call it “the special path”; its edges are called “the special edges”, its
internal nodes are called “the special nodes”. For brevity’s sake we denote on
the special path the direction from A to B as the direction “from left to
right”.
Next, let P and Q be two non-adjacent nodes of the special path: we
define “an external path between P and Q ” as a path between P and Q
of which no node of the special path is an internal node. If between a P
and a Q of the special path an external path exists, we call the special
nodes between P and Q —if P is to the left of Q : the special nodes
that are to the right of P and also to the left of Q— “covered” by that
external path.

It is clear that a covered special node can never be a candidate for C
an external path allows a path from A to B that bypasses all special nodes
it covers.
There are now two mutually exclusive cases: either there exist one or
more special nodes not covered b an external ath or each special node is
covered by at least one external path.
In the first case the theorem is true, for each path from A to B must
pass through all uncovered special nodes, hence any special node that is not
covered by any external path can be taken as node C.
In the second case the theorem is also true, for if each special node
is covered by at least one external path, two paths from A to B exist that
have no internal node in common. We shall show this existence by construction.
We shall construct a sequence of external paths PQ0, PQ1, ... , PQN .
The left-hand terminal node of the path PQi will be denoted by Pi ; the
right-hand terminal node of the path PQi will be denoted by Qi . Denoting
the relation “to the left of” by “<” our sequence of external paths shall
have the following two properties:
Property 1:

A = P0 < P1 < Q0 ≤ P2 < Q1 ≤ P3 < Q2 ≤ ..... QN-2 ≤ PN < QN-1 < QN = B .

Property 2:

For i ≠ j the external paths PQi and PQj have no internal node in
common .

Because (see property 1) we have for 0 < i < N

Pi < Qi-1 ≤ Pi+1 < Qi ,

Qi-1 and Pi+1 , if not coincident, can be connected via special nodes
between them on the special path and this entire connection will be covered by
PQi; similarly we connect A with P1 and QN-1 with B .
Then PQ0, PQ2, PQ4 ,... and PQ1, PQ3, PQ5 supplemented with the
connections introduced in the previous paragraph form two paths from A to
B that have no internal node in common: on account of property 1 they share

no special nodes, on account of property 2 they share no non-special nodes
as internal nodes.
For PQ0 we choose an external path with P0 = A —because the
leftmost special node is covered, such an external path exists— and Q0 as far
to the right as possible. If Q0 coincides with B the construction stops
here, otherwise we proceed repeatedly as follows until an external path PQN
with QN = B has been selected.
Let PQi be the last external path selected and let Qi not coincide
with B . Then Qi is a special node and, hence, covered by at least one
external path. For PQi+1 we choose a path covering Qi with Qi+1 as far
to the right as possible. For i = 0 , the fact that Q0 < Q1 and the way
in which PQ0 has been selected imply P0 < P1 ; for i > 0 , the fact
Qi < Qi+1 and the way in which PQi has been selected imply that the path
PQi+1 does not cover Qi+1 and hence we conclude Qi-1 < Pi+1, The
quality Pi+1 < Qi follows from the fact that PQi+1 covers Qi . This
proves the inequalities mentioned in property 1 . Property 2 follows from
the fact that PQi+1 has no internal node in common with PQk for 0 ≤ k ≤ i :
such a common node would provide an external path between Pk and Qi+1 , the
existence of which is not compatible with the construction of Qk as a
rightmost node. And this completes our proof.
Acknowledgements. I am indebted to C.S.Scholten for having found the theorem,
to Alain Martin for having screened my proof, and to my mother, mrs.B.C.Dijkstra -
Kluyver for having discovered a serious omission in my proof’s first
presentation —as a matter of fact, the omission was so serious that she did not
believe the proof— .
Note. With directed paths A → B and P ⇒ Q the proof can be applied directly
to the more interesting case of directed graphs.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENEN iBurroughs Research Fellow

The Netherlands.

Transcribed by Martin P.M. van der Burgt

Last revision

2015-01-25

.
