---
title: "Bulterman’s theorem on shortest tree (EWD1131)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1131.html
published: "1992-07-20T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1131.PDF
---

# Bulterman’s theorem on shortest tree

EWD 1131

Bulterman’s theorem on shortest tree

We consider a finite complete graph in which each edge has a length; as usual, the length of a tree is defined as the sum of the lengths of its edges. The shortest subspanning tree of such a graph is not necessarily unique. At the last meeting of the ETAC, however, Ronald W. Bulterman pointed out that all shortest subspanning trees give rise to the same bag of edge lengths. Here is a proof.

*         *         *

Lemma   A subset of nodes that in the complete graph can be connected by edges of minimum length only, is so connected in a shortest subspanning tree.

Proof   Consider for a given subspanning tree a nonextensible set Q of nodes, connected in the tree by edges of minimum length only. Let Q be a true subset of P, a set of nodes that can be connected in the complete graph by edges of minimum length only. The lemma is then proved by constructing a subspanning tree shorter than the given one.

Q being a true subset of P, there exists an edge from a node in Q to a node in P-Q and of minimal length. Because Q is nonextensible in the given tree, adding that edge to the given tree closes a cyclic path that contains a longer edge. Removal of the latter yields a tree that is shorter than the given tree. (End of Proof.)

This lemma gives rise to a (for me at least) new algorithm for shortest subspanning trees. To begin with we only consider the edges of minimum length; they partition the vertices in connected components —and because at least 1 component contains at least 2 vertices, there are fewer components than the original graph has vertices—. To construct the shortest subspanning tree we first select for each component of p vertices p-1 edges of minimum length that form a subspanning tree for the p vertices of that component. The remaining edges are produced as solution of the shortest tree problem for a reduced graph. The above components are the vertices of the reduced graph, whose edges are the shortest edges connecting any pair of components.

Because the number of minimum length edges and the reduced graph are both only dependent on the partitioning and hence independent of how the nondeterminacy is resolved, all shortest subspanning trees yield the same bag of edge lengths. And this concludes my proof of Bulterman’s theorem.

The theorem is not surprising at all: it holds in the extreme cases —all edge lengths different and all edge lengths equal— and for sufficiently incommensurable edge lengths I guess that additive decompositions are unique anyhow. This note has been written because the proof is very satisfactory: it disentangles precisely what has to be disentangled.

Nuenen, 20 July 1992

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
27-Nov-2011
