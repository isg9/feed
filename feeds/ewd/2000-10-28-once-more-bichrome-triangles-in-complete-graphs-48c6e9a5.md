---
title: "Once more bichrome triangles in complete graphs (EWD1302)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1302.html
published: "2000-10-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1302.PDF
---

# Once more bichrome triangles in complete graphs

We consider the bichrome triangles in a complete n-graph of which each of its edges is either red or blue. Each bichrome triangle has 2 "mixed Vs," where a mixed V is a pair of edges with different colours and a shared end point, while a monochrome triangle has no mixed Vs. Thus we can find an upper bound for nbt, i.e. the number of bichrome triangles, by halving an upper bound for the number of mixed Vs. We distinguish even and odd n.

n = 2k  .   The n−1 edges that share an end point form at most k∙(k−1) mixed Vs. Hence there are at most n∙k∙(k−1) mixed Vs, and

(0)
nbt ≤ k2∙(k−1)     .

n = 2k + 1  .  The n−1 edges that share an end point form at most k2 mixed Vs. Hence there are at most k2∙n mixed Vs and

(1)
nbt ≤ k2∙(2k+1)/2     .

(Note that the right-hand side need not be integer.)

The rest of this note is devoted to the question whether these upper bounds are as low as possible. The answer will turn out to be affirmative. This will be demonstrated by constructing for arbitrary n a graph such that for each node red and blue degree are as equal as possible, i.e. the situation from which the upper bounds (0) and (1) are derived [the red/blue degree of a node is the number of red/blue edges sharing that node as an endpoint.] We deal with even and odd n separately.
n = 2k   Partition the n nodes into k A-nodes and k B-nodes, colour the AA- and the BB-edges red and the AB-edges blue. Then each node has a red degree equal to k−1 and a blue degree equal to k, which, n−1 being odd, is as equal as you can get them.

n = 2k + 1  This case is slightly more complicated because the situation that in each node the degrees are both equal to k can only be obtained for even k; for uneven k we must admit 1 node with degrees (k−1, k+1); this corresponds to the situations in which the right-hand side of (1) is not an integer.

As before we partition the n nodes, but this time into n+1 A-nodes and n B-nodes. Were we to colour them as before, each A-node would have degrees (k,k) as desired, but each B-node would have degrees (k−1,k+1). To remedy the situation, consider the following transformation:

A

—

A

A

—

A

|

|

→

|

|

B

B

B

B

While it leaves the degrees of all A-nodes unchanged, it can be used to change at 2 B-nodes degrees (k−1, k+1) into (k,k), as desired. With k = 2∙h or k = 2∙h+1, we can apply this transformation on h disjoint pairs of B-nodes (and h distinct, and hence initially red, AA-edges).

Note For odd k, a node with degrees differing from (k,k) is unavoidable because the sum of the red/blue degrees is even.

(End of Note.)

Austin, 29 October 2000

Prof. Dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

Transcribed by Jordan Olsommer.

Last revised Sat, 24 Nov 2007.
