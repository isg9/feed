---
title: "An analytical proof of the Butterfly Theorem (EWD866)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD866.html
published: "1983-11-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD866.PDF
---

# An analytical proof of the Butterfly Theorem

In a circle with centre O the chords AD and BC intersect one another in Q. The line PQR intersects AB in P and CD in R and is orthogonal to OQ. Prove PQ = QR.

(The name “Butterfly Theorem” is derived from the shape of the quadrilateral ABCD; Q, however, need not lie inside the circle.)

Proof. Take for the diameter OQ the x-axis and let its ends be (–1,0) and (+1,0). Since the lines connecting A to those endpoints are orthogonal to each other, A can be characterized with some α as the point of intersection of the lines

α ⋅ (x + 1) + y = 0   and   (x – 1) – α ⋅ y = 0

and B, C, and D similarly with some β, γ, and δ respectively. (See Note.)

By multiplying the first equation by β and subtracting the second one we get

α ⋅ β ⋅ (x + 1) + (α + β) ⋅ y – (x – 1) = 0
(0)

By virtue of its construction (0) is a line through A; being symmetrical in α and β, it is also a line through B; hence, (0) is the equation of the chord AB. And similarly for the other chords.

The fact that Q with coordinates (q,0) lines on AD and on BC yields therefore

α ⋅ δ ⋅ (q + 1) = q – 1
(1)

β ⋅ γ ⋅ (q + 1) = q – 1
(2)

and since we may confine our attention to the case

q + 1 ≠ 0
(3)

— because, with Q on the circle, the theorem is trivial —

α : β = γ : δ
(4)

On account of (0), the y-coordinate p of P satisfies

(α + β) ⋅ p = (q – 1) – α ⋅ β ⋅ (q + 1)       , or, on account of (1),

(α + β) ⋅ p = (δ – β) ⋅ α ⋅ (q + 1)
(5)

On account of (0), the y-coordinate r of R satisfies

(γ + δ) ⋅ r = (q – 1) – γ ⋅ δ ⋅ (q + 1)      , or, on account of (2)

(γ + δ) ⋅ r = (β – δ) – γ ⋅ (q + 1)
(6)

On account of (4), (5) and (6) are the same equation except for a change of sign. (The case α + β = 0 and γ + δ = 0 corresponds to the degenerate case that both AB and CD are perpendicular to OQ.)

Note. For brevity’s sake we allow ourselves an α that may be “infinity”, instead of writing

α0 ⋅ (x + 1) + α1 ⋅ y = 0       and       α1 ⋅ (x – 1) – α0 ⋅ y = 0

(End of Note.)

(End of Proof.)

This theorem was shown to me on 27 Oct 1983 at UT at Austin by Shang-Ching Chou, who had proved it mechanically. The theorem was new for me and the proof had been long (I don’t remember, how long). In most of his proofs, coordinates of points had been chosen as free parameters. On my flight from Austin to Los Angeles I realized that characterizing A by α as done above would probably come in handy since the only thing done with A,B,C, and D was drawing chords through them: the sides of the quadrilateral are of more interest than its vertices.

The Tuesday Afternoon Club assisted me with the first draft of this proof. It was a nice exercise in analytical geometry.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

14 November 1983

prof.dr.Edsger W. Dijkstra

Burroughs Research Fellow

transcribed by Jonathan David Arndt

revised
17-Dec-2019
