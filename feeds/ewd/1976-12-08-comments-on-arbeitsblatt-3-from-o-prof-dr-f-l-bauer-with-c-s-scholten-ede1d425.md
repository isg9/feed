---
title: "Comments on Arbeitsblatt 3 from o.Prof.Dr.F.L.Bauer (with C.S.Scholten) (EWD602)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD602.html
published: "1976-12-08T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD602.PDF
---

# Comments on Arbeitsblatt 3 from o.Prof.Dr.F.L.Bauer (with C.S.Scholten)

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Comments on Arbeitsblatt 5 from o.Prof.Dr.F.L.Bauer.

by Edsger W.Dijkstra and C.S.Scholten
For the integer function m(x, y) of the integer arguments x and y , given by

m(x,y) = (x ≥ y → x+1 ▯ x < y → y-1)(1)

it is required to prove that

(x = y → y+1 ▯ x ≠ y → m(x, m(x-1, y+1))) = m(x, y)(2).

Proof. From (1) follows that the first alternative in the left-hand side of
(2) may be replaced by x = y → m(x, y) ; the proof is complete when we show
that the second alternative can be replaced by x ≠ y → m(x, y) . This letter
replacement is allowed as we shall prove

m(x, m(x-1, y+1)) = m(x, y)(3)

From (1) follows

X ≥ y ⇒ (m(x, x) = m(x, y))(4)

From (1) it also follows that the only possible values of m(x-1, y+1) are
x and y . If m(x-1, y+1) = y , (3) holds trivially. Otherwise we have
m(x-1, y+1) ≠ y , which implies —on account of (1)—

m(x-1, y+1) = x and x-1 ≥ y+1 .(5)

As (5) implies x ≥ y , we conclude by (4) and (5) that (3) then holds as
well. (End of proof.)
The above proof has been given, because 33(!) lines of formal text, as
dedicated to a proof of this theorem in said Arbeitsblatt 3, is to our tastes
a bit unwieldy. We appreciate that the proof in said Arbeitsblatt 3 has been
conducted in terms of a limited repertoire of operations that seem suitable
for mechanization. When, however, the length of such proofs and their obvious
tedium are presented —as is often the case— as conclusive evidence for the
necessity of such mechanizations, it should be clear that at least the above
example has failed to convince us of such necessity.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

NL-4565 NUENENBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-01-24

.
