---
title: "A methodological sequel to EWD771 (EWD772)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD772.html
published: "1980-12-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD772.PDF
---

# A methodological sequel to EWD771

EWD772 -
0

A methodological sequel to EWD771.

The first time I heard the problem mentioned in EWD771 it was only required to establish the existence of at least 1 homogeneous triangle, and the proposed proof was as follows.

Consider an arbitrary node, say X. Among the five edges meeting at X, one of the two colours, say red, dominates. Let XP, XQ, and XR be three red edges. Either the triangle PQR has a red side, PQ say, or it has no red side. In the first case triangle XPQ is homogeneously red; in the latter case triangle PQR is homogeneously blue.

Before I found the proof recorded in EWD771, I found the following argument for establishing the existence of a second homogeneous triangle.

We know the existence of one homogeneous triangle. Let it be red; call its angles the A's and denote the remaining three nodes by the B's. Then there exists a homogeneously red triangle AAB or in each B at most 1 red AB-edge meets.

In the latter case, at least 2 blue AB-edges meet at each B. But then BBB is homogeneously

EWD772 -
1

red or BBB has a blue edge. In the last case there exists a homogeneously blue triangle of the type BBA, since in both end points of the blue BB-edge two blue AB-edges meet, which have at least one A in common.
I never liked that proof --I don't think I ever recorded it before-- . I disliked it for its case analysis, which distinguishes among three cases: the first and the second homogeneous triangle share 0, 1, or 2 nodes. The proof of EWD771 is so much nicer because it avoids this case analysis; also it does not distinguish between a first and a second triangle. Finally it avoids the (minor?) ugliness of the "say red".
The trouble started already with "Consider an arbitrary node, say X.", thereby destroying the symmetry among the nodes. In the second part we have, similarly, the A's versus the B's. (We could have made it still worse by giving the A's and the B's subscripts!) What we have lost, right at the start, is the symmetry between the colours.

To my feeling, the underlying "secret" of EWD771's effectiveness is that it has fully exploited the colour symmetry: for a pair of edges, being of equal or different colour

EWD772 -
2

are the only functions that are invariant under colour inversion. Maintaining the symmetry among the nodes --to the extent that in EWD771 they remain unnamed!-- did the rest. A final virtue may be that the proof of EWD771 is less of an invitation to visualization. To quote E.T. Bell (from his essay on Cayley and Sylvester in "Men of Mathematics")
"If there is any virtue in talking about situations which arise in analysis as if we were back with Archimedes drawing diagrams in the dust, it has yet to be revealed. Pictures after all may be suitable only for very young children; Lagrange dispensed entirely with such infantile aids when he composed his analytical mechanics. Our propensity to "geometrize" our analysis may only be evidence that we have not yet grown up."

Plataanstraat 5

5671 AL NUENEN

The Netherlands

30 December 1980

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Mon, 30 Jun 2003.
