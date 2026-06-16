---
title: "The chessboard covered with dominoes (EWD1306)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1306.html
published: "2001-02-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1306.PDF
---

# The chessboard covered with dominoes

EWD 1306

The chessboard covered with dominoes

We consider a partitioning of the squares of the 8×8 chessboard into 32 pairs of adjacent squares and call a pair of squares in the same row (column) “a horizontal (vertical) domino”. Then the number of horizontal dominoes with a black left square equals that of those with a white left square. [Corollary: the number of horizontal dominoes is even.]

Proof   Consider for a given partitioning one of the seven lines that separate two adjacent columns; it divides the board into a left and a right part. We observe that (i) because the column length is even, each column, and hence the left part consists of equal numbers of black and white squares, and (ii) because domino consists of a black and a white square, the dominoes entirely to the left of the line comprise equal numbers of black and of white squares. Combining the two observations, we conclude about the horizontal dominoes crossed by the line —i.e., the dominoes with 1 square at either side of the line— that there are as many with a black left square as with a white one. Since each horizontal domino is crossed by exactly one of the seven lines, we now can conclude by summation that the total number of horizontal dominoes with a black left square equals the total number of those with a white left square.

(End of Proof)

Note that of the chessboard dimensions, we only used that the height is even.

Acknowledgement   I owe thanks to Jayadev Misra who inspired me to write this down and to Jan L.A. van de Snepscheut, who contributed part of the argument.

Austin, 5 February 2001

prof. dr. Edsger W. Dijkstra

Department of computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

Transcription by Xiaozhou (Steve) Li.

Last revision: Wed, 4 May 2016.
