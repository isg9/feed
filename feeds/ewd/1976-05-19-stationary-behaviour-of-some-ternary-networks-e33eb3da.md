---
title: "Stationary behaviour of some ternary networks (EWD624)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD624.html
published: "1976-05-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD624.PDF
---

# Stationary behaviour of some ternary networks

EWD624 - 0

Stationary behaviour of some ternary networks.

We consider a graph of N vertices in which each vertex has a multiplicity three, i.e. in which three edges meet at each vertex. Because the number of edges equals 3N/2, we conclude that N must be even.

Each edge connects two different vertices -- i.e. no "auto-cycles"--; the graph is partially directed, more precisely: each vertex has an outgoing edge, an undirected edge, and an ingoing edge. (Such graphs exist for all even N ≥ 4.)

In the initial situation, 3N numbers -- which can be assumed to be all different from each other -- are placed in the vertices, three at each vertex. A move consists of sending for each vertex:

1) its maximum value to the neighbour vertex at the other end of its outgoing edge,

2) its medium value to the neighbour vertex at the other end of its undirected edge,

3) its minimum value to the neighbour vertex at the other end of its ingoing edge,

4) and of accepting three new values from its neighbours.

(We can also view a move as 3N/2 simultaneous swaps of values at the end of each edge.)

After the move, again three values are places at each vertex, and, therefore, a next move is possible. We are interested in the periodic travelling patterns as will occur in infinite sequences of moves.

Suppose that, before distributing the 3N values among the vertices, we had painted the N largest values red, the N smallest values blue, and the N remaining values in between white; then we are interested in final patterns in which at each vertex a red, a white, and a blue value can be found. Note that such a distribution of colours is stable: in each move two white values will be swapped along each undirected edge, and along each directed edge a red and a blue value will be swapped -- the red one will go in the direction of the arrow, the blue one will travel in the opposite direction -- ; after the move, again all three colours will be present in each vertex.

EWD624 - 1

We furthermore require that the period of the stationary behaviour is exactly N moves. Below we shall give constructions of such networks for each N ≥ 4 with the property that the desired stationary behaviour as described above will be established after a finite number of moves, independently of the initial distribution of the 3N values. The cases N = 4Z and N = 4Z + 2 are treated separately.

N = 4Z .

The directed edges form a single directed cycle; the 2Z undirected edges connect the pairs of in this directed cycle diametrically opposite vertices. (If the vertices are numbered from 0 through N-1 , then a directed edge goes from vertex nr.i to vertex nr.(i+1)mod N , and undirected edge connects vertex nr.i and vertex nr.(i+2Z)mod N .)

Proof of stabilization. Let k be the maximum value, such that the k largest values are all placed in different vertices; initially we have 1 ≤ k ≤ N . We shall first show that within a finite number of moves, k = N by showing that, if k
For reasons of symmetry, eventually each vertex will also have exactly one blue value. But when both red and blue values are evenly distributed among the vertices, so will the white ones be. Hence the stable state will have been reached. The period of the cyclic behaviour obviously equals N. (End of proof of stabilization.)

EWD624 - 2

N = 4Z + 2 .

Here the directed edges of the graph form two cycles of length 2Z+1 each. The 2Z+1 undirected edges each connect one vertex of the one cycle with one vertex of the other cycle. (Note that the way in which each vertex of the one cycle is paired with exactly one vertex of the other cycle, is arbitrary.)

Proof of stabilization. Let k be defined as in the previous proof and assume k
The above problem and solution emerged during my "Tuesday afternoon discussion" of May 17, 1977, with Feijen, Prins, Peeters, Martin, and Bulterman. It was Feijen who posed the problem as a generalization of the binary network -- without undirected edges -- that I had shown in my lectures that morning. The solution has been recorded because we liked the argument, in spite of the fact that it is far from giving sharp upper bound on the number of moves needed.

Plataanstraat 5

5671 AL NEUNEN

The Netherlands

prof. dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcribed by Mikhail Esteves.
Last revised on Fri, 1 Aug 2003.
