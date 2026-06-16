---
title: "A class of simple communication patterns (with C.S.Scholten) (EWD643)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD643.html
published: "1974-10-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD643.PDF
---

# A class of simple communication patterns (with C.S.Scholten)

Copyright Notice

The following manuscript

EWD 643: A class of simple communication patterns (with C.S.Scholten)

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 334–337 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective, Springer-Verlag, 1982. ISBN 0–387–90652–5.

Reproduced with permission from Springer-Verlag New York.
Any further reproduction is strictly prohibited.

EWD 643

A Class of Simple

Communication Patterns

Written in Conjunction with C.S. Scholten

We consider a finite, undirected graph each node of which contains a process. Processes contained in nodes directly connected by an edge of the graph are called each other’s neighbours.

An act of communication is only possible between two neighbours. At any moment in time each process is ready to communicate with precisely one of its neighbours; the act of communication between two neighbours can only take place when each of them is ready to communicate with the other, and, as soon as they are both ready to communicate with the other, the actual communication is assumed to take place within a bounded period of time.

For each node there exists an (otherwise arbitrary) cyclic order of its neighbours, and the act of communication with one of its neighbours causes the node to become ready to communicate with its next neighbour, where “next” is to be understood in terms of that cyclic order. It is this rigid rule of the locally cyclic communication patterns that justifies the word “simple” in the title of this note. For such systems we shall determine the conditions characterizing the absence of the dangers of deadlock or starvation.

We represent the state of each process by the presence of one arrow from its node towards (the node of) the neighbour it is ready to communicate with: hence each node has always one outgoing arrow along one of the edges of the original undirected graph. In this representation, the act of communication between two neighbours takes place when they point to each other; the act of communication causes a “rotation” of both outgoing arrows. In this representation, the absence of deadlock is equivalent to the existence of at least one edge along which two arrows (in opposite directions) are present.

Let c be an arbitrary cycle of the undirected graph, in which neither a node nor an edge occurs more than once. (Such cycles contain at least 3 different nodes.) On this cycle we choose an arbitrary direction, which gives each node a “right-hand” neighbour and a “left-hand” neighbour in the cycle. Because such cycles contain at least 3 nodes, these two neighbours are different. For the outgoing arrow of a node x of that cycle we define a “signature with respect to c”: if it points to a node that, in the cyclic order associated with x, lies in the range from (and excluding) the left-hand neighbour of x to (and including) the right-hand neighbour of x we call the arrow positive; otherwise we call the arrow negative.

Lemma 1.   No act of communication changes the truth-value of the predicate: the outgoing arrows of the nodes of the cycle c have the same signature with respect to c.

Proof. The value of the predicate can only change when the signature of the outgoing arrow of a node c is changed. This can only happen at an act of communication with either its left-hand, or its right-hand neighbour in the cycle c. This is only possible when two communicating neighbours on the cycle had outgoing arrows of different signature. The act changes the signature of both arrows, so their signatures remain different from each other. In short: if the predicate is false it remains false in spite of the possibility of changing signatures, if it is true, it remains true because none of the signatures can change. (End of proof.)

Lemma 2.   The existence of a cycle c with outgoing arrows all of the same signature, causes local deadlock and, if the original graph is connected, total deadlock.

Proof. None of the outgoing arrows of the nodes of c can have its signature changed, hence for each node of c the number of acts of communication it can perform is bounded (by a bound lower than the number of its neighbours). By induction the number of acts of communication of any node that is connected to c via a finite path, is bounded. (End of proof.)

Lemma 3.   In the case of total deadlock there is at least one cycle with all its outgoing arrows of the same signature.

Proof. Total deadlock means that no process has its outgoing arrow “matched” by an arrow in the opposite direction. Starting at any node, the step that consists of going from that node to the node its outgoing arrow points to can be repeated indefinitely. On a finite graph we must visit a node visited before, and hence a cyclic path (of at least 3 nodes) must exist: but that is a cycle with all its outgoing arrows of the same signature. (End of proof.)

Combining lemmas 2 and 3 we conclude our main

Theorem.   In the systems considered the absence/certainty of deadlock is equivalent to the absence/presence of at least one cycle of uniform signature.

Lemma 4.   A deadlock-free system remains deadlock-free when, at a moment that there are no arrows along a certain edge, that edge is removed, provided at both its ends the cyclic order of the remaining neighbours remains the same.

Proof. The removal of an edge does not create new cycles. Because at both ends the cyclic order of the remaining neighbours remains the same, the definition of the signature of arrows with respect to the remaining cycles is not changed. Hence the assumed absence of cycles with outgoing arrows of uniform signature therefore remains. (End of proof.)

*         *         *

Our lemmas and theorem remain valid in a more general setting. We have assumed that each process would be ready to communicate with its neighbours “in some cyclic order”. We have used that assumption only for two conclusions:

(1)

that contacts with left- and right-hand neighbours —i.e. the pair of neighbours on a cyclic path through the node in question— would alternate;

(2)

that each node will be ready to communicate with any of its neighbours within a bounded number of contacts.

When all local communication patterns satisfy properties (1) and (2), our conclusions remain valid provided we redefine the signature of an outgoing arrow of a node on a cycle c as follows: the arrow is positive if it points to the right-hand neighbour or will do so before pointing to the left-hand neighbour, the arrow is negative otherwise. These more general communication patterns are still “simple” in the sense that permanent nonactivity of a specific process will lead after a bounded number of communication acts to nonactivity of the whole network connected to it. Such networks are simple because the absence of the danger of deadlock implies then the absence of the danger of individual starvation.

For the sake of completeness we formulate

Lemma 5.   Consider a deadlock-free network with “a leaf”, i.e. a node with only one neighbour. If the leaf, together with its outgoing arrow, is removed at a moment that its neighbour did not point to it, and the cyclic order of its neighbor’s remaining neighbours remains the same, the resulting system is again deadlock-free.

Lemma 5 is a variation of Lemma 4, and we leave its proof to the reader.

*         *         *

The theorem described and proved in this note is a theorem of the type the need of which I discussed last month at lunch with C.A.R.Hoare, when we met in Newcastle-upon-Tyne. At the end of that discussion we agreed that the discovery of a class of such theorems might be a proper thesis topic. Is the moral of this note that that topic might be unsuitable, because too small?

The theorem given in this note and its proof have been inspired in particular by the self-stabilizing systems designed earlier by L. Lamport and C.S. Scholten, in which processes at the nodes of a tree were considered. A discussion with C.S. Scholten on the topic of EWD642 (still in statu nascendi) was the incentive for its discovery.

Nuenen

prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Corrado Cantelmi

edited by H. Richards for consistency with the published version

revised
23-Dec-2011
