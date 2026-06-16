---
title: "When messages may crawl, II (A sequel to EWD708) (EWD710)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD710.html
published: "1979-05-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD710.PDF
---

# When messages may crawl, II (A sequel to EWD708)

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

When messages may crawl, II (A sequel to EWD708).
In EWD708 I wrote “the messages are essentially empty envelopes;
allowing the envelopes to contain something was felt to be a minor
complication that we could postpone for the time being”. The purpose of this
note is to investigate how minor or major this complication turns out to be.
We modify the program by distinguishing the different messages from
C to D by a subscript i and the ones in the opposite direction by a
subscript j :

(1)

{P} do B1Ci,→ S1i; S1Di {P}

▯ B2Dj → S2Dj; S2Cj {P}

od

and its correctness proof now comprises the (many) theorems

P and B1Ci ⇒ wp(“S1Ci; S1Di”, P)(2i)

P and B2Dj ⇒ wp(“S2Dj; S2Cj”, P) .(3j)

In this note we regard the way in which the difference between
different messages from C to D has been indicated by the implementation as
irrelevant: the contents of the messages may differ, they may be sent along
different channels that can be distinguished, or any mixture thereof. It is
an implementation detail, from which we are allowed to abstract.
In analogy to EWD708, the following program is regarded to be
representative for the corresponding mail system:

{P} (A j: cj:= 0); (A i: di:= 0); {M}

do B1Ci → S1Ci ; di:= di + 1 {M}

▯ di > 0 → di:= di - 1; S1Di {M}

▯ B2Dj → S2Dj; cj:= cj + 1 {M}

▯ cj > 0 → cj:= cj - 1; S2Cj {M}

od

with the corresponding proof obligations

(E j: cj ≠ 0) or (E i: di ≠ 0) or P = M(4)

M and B1Ci ⇒wp(“S1Ci; di:= di +1”, M)(5i)

M and di > 0 ⇒ wp(“di:= di - 1; S1Di”, M)(6i)

M and B2Dj ⇒ wp(“S2Dj; cj:= cj + 1”, M)(7j)

M and cj > 0 ⇒ wp(“cj:= cj - 1; S2Cj”, M)(8j)

With the same notational convention —(9) and (10)— from EWD708, we
can satisfy (4), (6i), and (8j) by choosing for M

M:
wp(“(A i: S1Di ↑ di); (A j: S2Cj ↑ cj)”, P)(11)

provided that (11) makes sense. In analogy to EWD708 this implies
(A j: cj ≥ 0) and (A i: di ≥ 0) ,
but this invariance is duly maintained by the above representation of the
mail system. But for (11) to make sense, we must require that (A i: S1Di ↑ di)
and (A j: S2Cj ↑ ci) are defined, i.e. we must require

(A i1,i2: "SlDi1 ↑ di1; SlDi2 ↑ di2" = "S1Di2 ↑ di2; S1Di1 ↑ di1")(16)

and
(A jl,j2: "S2Cj1 ↑ cj1; S2Cj2 ↑ cj2" = "S2Cj2 ↑ cj2; S2Cj1 ↑ cj1")(17)

(This curious way of numbering results from my desire to give similar
formulae in this note and in EWD708 the same number.)
Salvo errore et omissione I have come to the conclusion that properties
(12) and (13) are stronger than necessary, and so are their consequences (14)
and (15), as mentioned in EWD708. In order to prove (5i) from (2i) the
weaker assumptions

(A j: B1Ci ⇒ wp("S2Ci ↑ ci", BlCi)) (14)

and
(A j: BlCi and wp("S2Cj ↑ cj; S1Ci", R) ⇒ wp("S1Ci; S2Cj ↑ cj", R))
(15)

suffice. (The absence of the beginning term “E1C” in (13) and (15) of
EWD708 is, in retrospect, just an omission.)
One way of proving (14) and (15) is to show —as suggested in EWD708—
for each (i,j)-pair

B1Ci ⇒ wp(“S2Cj”, B1Ci)
(12)

B1Ci and wp(“S2Cj; S1Ci’, R) ⇒wp(”S1Ci; S2Cj“, R) .(13)

Properties (12) and (13) have the attraction that they are local
properties of node C of the telex system. Note that relations (14) and
(15) are also a consequence of

B1Cj ⇒ (cj = 0) .(18)

Similarly, in the spirit of EWD708, we would try to prove for any
(j1,j2)-pair

”S2Cj1; S2Cj2“ = ”S2Cj2; S2Cj1“(19)

from which (17) follows. Again we should note the existence of the
alternative: cj1 = 0 or cj2 = 0 .
Summarizing our proof obligation with respect to node C , we have to show:

(A i, j; BlCi ⇒ wp("S2Cj, BlCi) or cj = 0)(20)

(A i, j: BlCi and wp("S2Cj; S1Ci", R) ⇒ wp("SlCi; S2Cj", R) or cj = 0)(21)

(A jl,j2: "S2Cj1; S2Cj2" = "S2Cj2; S2Cj1" or cj1 = 0 or cj2 = 0)(22)

With respect to node D we have, mutatis mutandis, the same obligations.
The weakening of our proof obligations, as represented by (20), (21),
and (22) leaves me with mixed feeling. Each time we have to make essential
use of the weakening, we have to prove that under certain circumstances
“exponents” are zero, and those are global properties that are not directly
reflected —at least for the time being I don’t see how— in the original telex
system. I console myself with the thought that, if a mail system is our
eventual target, they are not very desirable properties: in order to make
a mail system it is customary to make message receptions commute if possible.
One advantage of conditions (20), (21), and (22) is certainly that they give
the designer of a mail system a clear summary of his options.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENENBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-02-18

.
