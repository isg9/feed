---
title: "On a class of graphs with modest diameter (EWD987)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD987.html
published: "1986-10-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD987.PDF
---

# On a class of graphs with modest diameter

EWD 987

On a class of graphs with modest diameter

The diameter of a graph is defined as the minimum path length, measured in number of edges, that suffices to connect any two nodes. We confine ourselves to undirected graphs. (For the complete graph, the diameter equals 1, for a star it equals 2.)

For the n-dimensional “cube”, the diameter equals n, and so does the “fan” —number of edges meeting— of each node. Some time ago, Jan L.A. van de Snepscheut showed me how for n=3 the diameter could be reduced to 2 without changing the fan. Without proof he mentioned that under that constraint, the diameter could be almost halved (to n div 2 + 1). Looking for a constructive proof of that statement, I found another class of graphs with diameter almost n/3, and a smaller number of edges (2/3); the maximum fan, however, is n+1. More precisely:

for a graph of 2n nodes with n = 3�k + 2:

•     the diameter is k+3

•     half the nodes have a fan n+1

•     half the nodes have a fan k+2   .

The remainder of this note is devoted to a description of my construction.

*         *         *

As is to be expected, we identify our nodes by n-bit strings, parsed as a pair —called x—, followed by k triples —called the y’s—. From now on we shall call our nodes “strings”.

We only introduce connections between two strings that either

(i)only differ in their x,       or

(ii)   only differ in a single y.

The connection pattern for the 4 different x-values is given by

(The assignment of x-values to the four corners is immaterial; who wants to be specific may think of 00 at A, 11 at B, 01 and 10 each at a C-corner.) The important property of this graph is that each corner can be connected to each corner —i.e. including itself— by a path of length ≤ 3 and containing A, B, and at least one C-corner. Depending on its x, a string is called an A-string, a B-string or a C-string.

A pair of different y-values is an A-pair, a B-pair or a C-pair according to their bit-wise sum: for three values of the bit-wise sum (say 001, 010, and 100) it is an A-pair, for three other values (say 011, 101, and 110) it is a B-pair and for the last value (say 111) it is a C-pair.

Two strings, differing only in one y, have the same x, are thus both X-strings (with X = A, B, C); they are connected if and only if the two differing y’s form an X-pair.

The diameter of the resulting graph is k+3, as follows from the following algorithm for finding a path. Compare the y-components of the source string with the corresponding y-components of the target string. Wherever they differ, they form an A-pair, a B-pair, or a C-pair. In the worst case, they all occur. Choose in the x-graph a path from source x to target x containing nodes of all types needed: when the string is an X-string, all y’s where we found an X-pair can be transformed into their final value.

The A- and B-strings —together half of them— have a fan of 3�k + 3 = n + 1; the C-strings —the other half of them— have a fan k + 2. The total number of edges is therefore 2n-2�(n + k + 3).

Remark   We could have partitioned 7 bit-wise sums of two different y’s differently, e.g. 2 A-pairs, 2 B-pairs, and 3 C-pairs. For A-strings and B-strings the fan would have been 2�k + 3, for the C-strings 3�k + 2 = n. The total number of edges would have been 2n-2�(n + 2�k + 3). (End of Remark.)

Since the gain over the graphs mentioned by van de Snepscheut only materializes for larger n, and n occurs as exponent in the number of nodes, I do not expect my construction to be of any significance if nodes are machines. But it is nice to know that this construction exists, and it was fun to design it. (Describing it was just as hard as I had expected.)

Austin, 6 October 1986

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

United States of America

transcribed by Corrado Cantelmi

revised
15-Mar-2012
