---
title: "When messages may crawl (EWD708)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD708.html
published: "1979-03-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD708.PDF
---

# When messages may crawl

EWD 708

When messages may crawl.

The last session of the Tuesday Afternoon Club was devoted to the question when what has been proved to be correct when viewed as "a telex system", can also be implemented as "a mail system". The original motivation was the following.

We consider a network in which a node can transmit messages to (some) other nodes, and the only things we postulate about message transmission are

1) no message arrives before it has been sent

2) each message sent arrives eventually.

A system with such message transmission properties was called "a mail system".

The idea was to try to partition the correctness proof of a mail system in the following way:

1) prove that the system would be correct when viewed as "a telex system" in which message transmission is instantaneous, i.e. the sending and receiving of a message is part of the same "point action"
2) show that the system satisfies the antecedents of some general Laws —as yet unknown to us— stating when the correctness of a telex system implies the correctness of the corresponding mail system.

We spent the afternoon on pruning the problem and on collecting "evidence" by designing all sorts of telex systems allowing and not allowing replacement by mail transmission. On account of the evidence, some Laws were conjectured, some of these conjectures were refuted by counterexample, and finally a formal approach for tackling the problem was suggested, but by then I was too tired to pursue the matter any further. That is what I did the next day. Hence this note, which has been written with the hope of providing next week's session of the Tuesday Afternoon Club with an inspiring starting point.

*                  *

*

Consider the following repetitive program as representative for a two-node telex system

{P} do B1C → S1C; S1D {P}

∥ B2D → S2D; S2C {P}

od
(1)

Here the top line represents a telex message from node C to node D, the bottom line represents a telex message in the opposite direction. The guard B1C depends only on variables belonging to node C , in S1C only variables belong to node C are accessed, in S1D only variables belonging to node D, and similarly for the next line. (In this model, the messages are essentially empty envelopes; allowing the envelopes to contain something was felt to be a minor complication that we could postpone for the time being.) As indicated, P is the correspondent invariant relation. As part of the correctness proof of (1) the theorems

P and B1C ⇒ wp("S1C; S1D", P)
(2)

P and B2D ⇒ wp("S2D; S2C", P)
(3)

have been established.

We decided that the following program would be representative for the corresponding mail system

{P} c, d := 0, 0 ; {M}

do B1C → S1C ; d:= d + 1 {M}

∥ d > 0 → d:= d - 1 ; S1D {M}

∥ B2D → S2D ; c:= c + 1 {M}

∥ c > 0 → c:= c - 1 ; S2C {M}

od

Here we have to prove

(c, d) ≠ (0, 0) or P = M

(4)

M and B1C ⇒ wp("S1C; d:= d + 1", M)
(5)

M and d > 0 ⇒ wp("d:= d – 1; S1D", M)
(6)

M and B2D ⇒ wp("S2D ; c:= c + 1", M)
(7)

M and c &gt; 0 ⇒ wp("c:= c – 1; S2C", M)
(8)

Relations (4), (6), and (8) can be satisfied independently of (2) and (3) by a suitable choice for M. Defining for any statement S and any natural number x

S↑0 = skip
(9)

S↑(x + 1) = "S↑x ; S"
(10)

we can satisfy relations (4), (6), and (8) by choosing for M

M:      c ≥ 0 and d ≥ 0 and wp("S1D↑d ; S2C↑c", P)
(11)

(Note that "S1D ; S2C" = "S2C ; S1D": accessing disjoint sets of variables, they obviously commute.)

In the sequel we drop for the sake of brevity from (11) the (implied) terms c ≥ 0 and d ≥ 0 . Trying to prove (5) using (2) I discovered that I had to make two further assumptions about node C

B1C ⇒ wp(S2C, B1C)
(12)

wp("S2C ; S1C", R) ⇒ wp("S1C ; S2C", R) for any R
(13)

From (12) we immediately deduce for any x ≥ 0

B1C ⇒ wp("S2C↑x", B1C)
(14)

From (13) —which, for instance, would be satisfied if S1C and S2C were to commute— we immediately deduce for any x ≥ 0 and any R

wp("S2C↑x ; S1C", R) ⇒ wp("S1C ; S2C↑x", R)
(15)

In order to prove (5), we now proceed as follows.

M and B1C =
(by (11))

wp("S1D↑d ; S2C↑c", P) and B1C ⇒
(by (14))

wp("S1D↑d ; S2C↑c", P) and wp("S2C↑c", B1C =

wp("S1D↑d ; S2C↑c", P) and wp("S1D↑d ; S2C↑c", B1C) =

wp("S1D↑d ; S2C↑c", P and B1C) ⇒
(by (2))

wp("S1D↑d ; S2C↑c", wp("S1C ; S1D", P)) =

wp("S1D↑(d+1) ; S2C↑c ; S1C", P) ⇒
(by (15))

wp("S1C", wp("S1D↑(d+1) ; S2C↑c", P)) =
(by (11))

wp("S1C ; d:= d + 1", M)

In this derivation only the last equality is fishy, because our form for M is most definitely not a first-order formula, and yet we appeal to the axiom of assignment. Yet I am sure that it is correct ; the justification will require that the "variable" d does not occur in the original program (1). The 2nd and the 4th equalities depend on the disjointness of variables, the 3rd on the conjunction property of wp .

So much for the derived invariant relation. In order to complete the proof with a proof for convergence, we have to derive a new variant function as well, but that exercise is left for later.

In order to prove (7) from (3) we must make about node D the two assumptions to (12) and (13) about node C .

*                  *

*

The example is very modest, and in itself not very exciting. What does excite me is that the antecedent of our Law is formulated in local conditions only! Apparently the correctness proof of the telex system duly captures here all global aspects of the problem.

29th of March 1979

Plataanstraat 5

5671 AL NUENEN

The Netherlands
prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Sam Samshuijzen

revised Mon, 11 Jul 2005
