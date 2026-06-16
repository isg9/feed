---
title: "The non-unique node multiplicity (EWD1210)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1210.html
published: "1995-07-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1210.PDF
---

# The non-unique node multiplicity

For N ≥ 2, we consider a graph of N "nodes" with any pair of nodes being connected by an "edge" or not —in the jargon: an undirected graph without multiples edges and without autoloops—; the number of edges meeting at a node is called that node's "multiplicity".

Consider now two nodes. The difference between their multiplicities is determined by how they are connected to the other N-2 nodes, and is maximal if the one is connected to all other N-2 nodes, and the other to none of them. Hence, the difference between two multiplicities is at most N-2; hence there are at most N-1 different multiplicities, and hence, with N nodes, there are at least 2 nodes with the same multiplicity.

*                 *
*

I would like my readers to appreciate that the argument in the previous paragraph —which is only 74 words long— contains neither a case analysis, nor a reductio ad absurdum.

It has been written in reaction to P.R. Halmos's treatment of this problem, a treatment which I think very ugly. By assigning a special role to nodes with multiplicity 0, Halmos immediately destroys the symmetry between presence and absence of edges. He then embarks on a three-fold case analysis: no such node, 1 such node, or at least 2 such nodes. in the final argument, which is phrased for N=100 but is supposed to hold for N=99 as well, it is no longer clear where 100 ≥ 2 is used. Note that in the following quotation from Halmos, "to be acquainted with" is supposed to be a symmetric relation.

Suppose that a bunch of us are together in a room, 100 of us, say, and we form temporarily a small society of our own. In this closed society there are a certain number of acquaintanceships: some of us are acquainted with some others. I don't know which ones of us are acquainted with which others, but I'm sure of one thing: I'll bet that there are at least two of us that have the same number of acquaintances.

Believe it? Let's see if I can make it convincing. Suppose that someone asked us, each of us, myself included: "How many other people in this closed society are you acquainted with?" We could all tell him an answer, somewhere between 0 and 100. No, wait a minute. If there are exactly 100 of us, then nobody is acquainted with 100 OTHER people: the largest number can be no larger than 99. As fas as 0 is concerned, that's all right, there could well be some hermits among us, but it is not likely, and, in any event, I can easily settle that case. If there are two or more hermits, then I've already won my bet: any two hermits have the same number of acquaintances. If there is only one hermit, then let's ostracize him —let's not count him— let's go so far as to pretend that he isn't here. I must still prove that among the remaining 99 there are two of us with the same number of acquaintances, and I'll do so — but because 100 is easier to say than 99, let me assume that even if the possible hermit is not counted there are still 100 of us left.

So then, what possible numbers will each of the 100 of us give to the questioner? Answer: any number between 1 and 99 inclusive. What is it that I am betting? Answer: that two of us will give the questioner the same number. Indeed: how could we fail? There are only 99 numbers to tell him and there are 100 of us telling: there must be at least one repetition. (Quoted from [1].)

The shape of the above argument is crummy, its presentation is perhaps even worse. To turn the nodes into persons is no improvement, on the contrary, since the anthropomorphic terminology invited the introduction of a contradiction. On the one hand, Halmos the author belongs to the "100 of us", on the other hand, Halmos the mathematician makes a bet. In the latter capicity he states "I don't know which ones of us are acquainted with which others", in the former capacity he can tell the questioner the number of his acquaintances! In short, total confusion.

There is no point in blaming Halmos, for he is of an older generation and from a different culture; moreover, he is not obliged to pretend to be better than he is. What worries me is that a sizeable fraction of the mathematical community thinks his performance good enough. In [0], A. Dijksma writes in his "Laudatio" about Halmos: "Throughout his career, which now comprises more than 50 years, he has maintained a high standard. As a result he has become renowned as an educator and teacher […] brilliant expositor […]". The reader cannot help wondering what the old man is being praised for. Talking down to his audience?

We all know that the mathematical community feels threatened, but it should understand that it does not improve its intellectual standing by lowering its standards of rigour and clarity.

[0] Dijksma, A. "Paul R.Halmos: A Complete Professional Mathematician", Nieuw Archief voor Wiskunde, Vierde serie Deel 13 No.1 maart 1995, pp. 49-60

[1] Halmos, P.R. "To Count or to Think, That is the Question", Nieuw Archief voor Wiskunde, Vierde serie Deel 13 No.1 maart 1995, pp. 61-7

Nuenen, 14 July 1995

prof. dr. Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188
USA

transcribed by Joan Llosas Colongo
revised Wed, 2 Dec 2009
