---
title: "The chessboard covered with dominoes (EWD1306a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1306a.html
published: "2001-02-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1306a.PDF
---

# The chessboard covered with dominoes

EWD 1306a

The
chessboard covered with dominoes

Claim We consider a partitioning of the squares of a chessboard into 32 pairs of adjacent squares, and call each pair a domino. A domino whose squares are in the same row we call a horizontal domino. We claim that the number of horizontal dominoes with a black left square and the number of horizontal dominoes with a white left square are equal. [Corollary: the number of horizontal dominoes is even.]

Proof For a given partitioning, consider one of the seven lines that separate two adjacent columns; it divides the board into a left and a right part. We observe that

- because the column length is even, each column, and hence the left part, contains equal numbers of black and white squares, and

- because a domino consists of a black and a white square, the dominoes entirely to the left of the line comprise equal numbers of black and white squares.

Combining the two observations, we conclude that, among the horizontal dominoes crossed by the line, there are as many with a black left square as with a white one. Since each horizontal domino is crossed by exactly one of the seven lines, we conclude by summation that the number of horizontal dominoes with a black left square and the number of horizontal dominoes with a white left square are equal.

(End of Proof)

Note that of the chessboard dimensions, we only used that the height is even.

Acknowledgement I owe thanks to Jayadev Misra, who inspired me to write this down, to Jan L.A. van de Snepsheut, who provided a major part of the argument, and to Ham Richards, who contributed stylistic improvements.

Austin,
11 February 2001

prof. dr. Edsger W. Dijkstra

Department
of computer Sciences

The
University of Texas at Austin

Austin,
TX 78712 - 1188

USA

Transcription by James Lu.

Last revision: Sat, 24 Nov 2007.
