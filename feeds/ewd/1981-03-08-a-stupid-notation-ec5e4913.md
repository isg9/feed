---
title: "A stupid notation (EWD782)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD782.html
published: "1981-03-08T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD782.PDF
---

# A stupid notation

EWD 782

A stupid notation.
For a set of  n  elements, the number of different subsets of  k  elements (0 ≤ k ≤ n) is n!
––––––––

k!�(n-k)!
.   (0)

Mathematicians call each such subset "a combination of k out of n" and denote their number by
( n

k
)         ,   (1)

a formula of which they are not quite sure how to pronounce it; one sometimes hears "n above k". From (0) and (1) they then derive the theorem
(n
k ) = (n
n-k )   (2)
*         *         *
For a set of  i+j  elements (i≥0, j≥0), the number of different ways it can be partitioned into two subsets of  i  and  j  elements respectively is
(i+j)!
–––––

i! � j!
(3)

Let us call each such partitioning for short a "partitioning into i and j" and let us denote their number —in honour of Blaise Pascal— by
P(i,j)   (4)

which is, of course, a symmetric function of its arguments:
P(i,j) = P(j,i)   (5)
*         *         *
Notation (1) is bad in all respects I can think of. It is bad from (what computing scientists call) a lexical point of view: it does not fit on a line and —worse!— it usurps a parenthesis pair. Furthermore it has introduced such a novel arrangement of characters on paper that its pronunciation requires a completely new convention. (And these are objections I have learned not to think lightly of.)
It is also bad in that it destroys the symmetry: it represents a symmetric function of two arguments by an asymmetric function of their sum and one of the two arguments, thus forcing one to remember which is which. (This is no joke, for in older literature one can encounter instead of (1): C
k

n
.) Finally it raises the shallow (2) to the status of a "theorem"!
It is furthermore profoundly bad in that it is specifically geared to expressing the number of different partitions into 2 subsets of given sizes, whereas nothing prevents us from generalizing and defining for instance
P(i,j,h) =
(i+j+h)!
––––––––

i! � j! � h!
(6)

which is obviously as symmetric in all its arguments as it could be. It provides the means for a smooth formulation of
P(i,j,h) = P(i,j) � P(i+j,h)   (7)

which is a bit less insipid than the "theorem" (2).
*         *         *
After the above one might think that the dead horse does not need any more flogging, but I think I must disagree, because notations can have a more sneaky influence on our thinking habits than we usually care to admit.
Let us prove afresh that (3) gives the value of P(i,j). Consider the (i+j)! permutations of the i+j elements. With each permutation we associate a partitioning into  i  and  j  by separating the first  i  elements from the  j  remaining ones. Since i!�j! distinct permutations thus lead to the same partitioning into  i  and  j, P(i,j)�i!j! = (i+j)!, hence (3).
But this is not the proof of (0) that I was given a long time ago. That one went as follows. Let us choose  k  elements. For the first element we can choose among  n , for the second one among (n-1), and so on, until we choose the kth element among (n-k+1). Since each combination of  k  elements can thus be chosen in k! different ways, we conclude
(n
k ) = n�(n-1)�...�(n-k+1)
–––––––––––––––

k�(k-1)�...�1      ,

which formula was subsequently "cleaned up" by multiplying numerator and denominator by (n-k)!. Confronted with (4) as an alternative for (1), one of my colleagues —a statistician— defended (1) as being "closer to one's intuition"! I guess he was educated with the second proof.
Many mathematicians derive part of their self-esteem by feeling themselves the proud heirs of a long tradition of rational thinking; I am afraid they idealize their cultural ancestors.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

8 March 1981

prof. dr. Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Corrado Cantelmi

revised
20-Sep-2011
