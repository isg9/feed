---
title: "Jan van de Snepscheut’s tiling problem (EWD1198)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1198.html
published: "1995-02-03T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1198.PDF
---

# Jan van de Snepscheut’s tiling problem

Jan van de Snepscheut's tiling problem

When a 6×6 square is covered by 18 2×1 dominoes, each domino is "cut" into 2 unit squares by one of the (5 + 5 =) 10 grid lines. Show that there is a grid line that cuts no domino.

Proof   Define for each grid line its "cut frequency" as the number of dominoes it cuts. For an arbitrary grid line we observe

(i) it divides the 6×6 square into two rectangles, each of an even number of unit squares.

(ii) each domino it cuts covers in both rectangles 1, i.e. an odd number of unit squares.

(iii) each domino it does not cut covers in both rectangles an even number of unit squares

From (i), (ii), (iii) we conclude

(iv) cut frequencies are even.

Because 10 grid lines cut 18 dominoes, the average cut frequency equals 1.8. Hence —on account of the generalized pigeon-hole principle—

(v) the minimum cut frequency is ≤ 1.

From (iv) and (v) we conclude that the minimum cut frequency equals 0, which proves the theorem. (End of proof.)

*               *
*

A minor reason for writing down the above is the consideration that the homework I give to my undergraduate students should be done by me as well. In addition I would like to show them, by way of example, one of the ways in which I might present the argument.

The main reason for writing down this argument, however, is that it is shorter and simpler than what I remember having seen. I remember arguments concluding the existence of at least 1 non-cutter from the existence of at most 9 cutters, whereas the above argument avoids the partitioning of grid lines into cutters and non-cutters.

Finally, the above argument is a nice example of using the pigeon-hole principle in the form

the minimum ≤ the average.

Austin, 3 February 1995

prof.dr.Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188

transcribed by Alejandro Ruiz
revised Wed, 12 Nov 2008
