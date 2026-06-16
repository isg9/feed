---
title: "On one of Cayley's theorems (EWD677)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD677.html
published: "1978-08-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD677.PDF
---

# On one of Cayley's theorems

EWD677 -
0

On one of Cayley's theorems

A (finite) graph consists of a (finite) set of nodes with at most one edge between any two nodes. The so-called "multiplicity" of a node is defined as the number of edges of which that node is an endpoint.

A connected graph without cycles is called a "tree"; a tree with N nodes has N-1 edges. In a tree the nodes with a multiplicity =1 are called its "leaves". A tree with at least 2 nodes has at least 2 leaves.

In the following "a labeled tree" is a tree in which each node is labeled with a distinct integer. From a labeled tree with at least 2 nodes we can remove the leaf with the lowest number, together with the edge connecting it to the rest of the tree; the remaining graph is again a labeled tree. Hence this action can be repeated until the tree has been reduced to a single node; that remaining node is the one with the maximum number.

For instance, the tree

4
\
3 0 5
\ / /
7 - 1
/ \
2 6

would give rise to the following sequence - in the order from left to right - of edge removals; each time we have written the number of the leaf being removed in the upper line:

2 3 4 0 5 6 1
1 7 0 7 1 1 7

EWD677 -
1

We remark

1.)that the right-most value of the bottom line is always the highest node number

2.)that the top line is always a permutation of the remaining node numbers

3.)that the number of times that a value occurs in such a scheme equals the multiplicity of the corresponding node.

When we remove from such a scheme the top line and the right-most column --in our example
1 7 0 7 1 1

would be the result-- each value has been removed once (on account of 1. and 2.) and (on account of 3 the number of times that a value occurs in such a sequence is one less than the multiplicity of the corresponding nodes. Hence the leaves are the nodes whose number is missing in the sequence.
Given the bottom line minus the right-most element and knowing the set of integers used to label the nodes, we can therefore reconstruct the left-most element of the top line: it is the minimum value from the set that is missing in the sequence. Because for a tree of N nodes the sequence is N-2 elements long, at least 2 values are missing; therefore that minimum missing value never equals the maximum node number, i.e. the construction is possible for any sequence of N-2 node numbers.

The next value in the top line is found by the same argument after reducing the set of node numbers by removing from it the value just filled in, and reducing

EWD677 -
2

the sequence by the removal of its left-most element. By repeating the argument, the first N-2 elements of the top line can be reconstructed. Finally, of the right-most column the bottom element is reconstructable on account of observation 1. and its top element by observation 2. .

Hence, for a set of N labeled nodes, there is a one-to-one correspondence between the trees connecting these nodes and the sequences of N-2 node numbers. Because the number of such sequences is obviously equal to

NN-2 ,

this value also equals the number of possible trees connecting N labeled nodes. This is ascertained by one of Cayley's famous theorems, of which this note offers a new proof.

Remark. To tell the truth: this proof is not so very new, as I discovered it more than 20 years ago. It surfaced yesterday in a discussion at the Tuesday Afternoon Club, which urged me to devote a small note to it. Hence. (End of remark)

23rd of August, 1978

Plataanstraat 5

5671 AL NUENEN

The Netherlands

prof.dr. Edsger W. Dijkstra

BURROUGHS Research Fellow

Transcription by John C Gordon
Last revised on Tue, 24 Jun 2003.
