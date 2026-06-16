---
title: "On disjoint binary numbers (EWD1312)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1312.html
published: "2001-11-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1312.PDF
---

# On disjoint binary numbers

EWD 1312

On disjoint binary numbers

For a pair of natural binary numbers we define a "double" as a position in which both have a 1 , nd.(h,k) denotes the the number of doubles in the pair (h,k)   and   (h,k) is "double-free" means nd.(h,k)=0 . [In the title, the concept "double-free" is hinted at by the adjective "disjoint".]

Another way of stating that the pair (h,k) is double-free is s.(h+k) = s.h+s.k , where s.n equals the number of 1's in the binary representation of n. [We could define s by s.n = nd.(n,n).]

The rest of this note is inspired by a theorem of Jay Misra's about the odd/evenness of binomial coefficients. The link with the concept "double-free" is given by

(

n

k

) is odd ≡ (k,n-k) is double-free ,

a lemma we won't prove here.

Misra's theorem about binomial coefficients was (practically) equivalent to the following theorem about double-free pairs:

Theorem Consider for positive integers h and k the three pairs (h,k), (h−1,k) and (h,k−1) ; an even number of them is double-free.

*             *

*

Proof In the following it helps to remember that in binary, a decrease by 1 boils down to an inversion of the right-most 1 and all zeroes to its right; to the left of the inverted 1 , nothing is changed.

Because h and k are positive, each contains at least one 1, hence a right-most 1. Let h end with a 1 followed by x zeroes, and let k end with a 1 followed by y zeroes. Our theorem being symmetric in h and k , we can confine our attention to the situation x≤y . We now consider two main cases, viz. x=y and x<y .

x=y.    The pair (h,k) has at least x as a double, so nd.(h,k)>0 . Since both nd.(h−1,k) and nd.(h,k−1) both equal nd.(h,k)−1 —the double at x is destroyed and no new doubles are created— nd.(h−1,k)=nd.(h,k−1) . Hence zero or two pairs are double-free.

x<y    Since in the transition from (h,k) to (h−1,k) all the changes in h occur in positions where k has zeroes, we conclude nd.(h,k)=nd.(h−1,k). Since (h,k−1) has at least x as a double, nd.(h,k−1)>0 ; again zero or two pairs are double-free.

And this completes the proof.

*             *

*

If you want to prove this essentially combinatorial theorem from first principles, case analysis is all but unavoidable. In this problem, the challenge is to keep the number of cases small. Before the above, I had designed two proofs which each distinguished five different cases, in one of them the main distinction being 0, 1 or more doubles in (h,k). Thanks to the introduction of the function nd , the specific value of nd.(h,k) need not be mentioned.

Austin, 11 November 2001

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Daniel Kolditz Rubin

revised Wed, 14 Nov 2007
