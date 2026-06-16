---
title: "The argument about the arithmetic mean and the geometric mean, heuristics included (EWD1171)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1171.html
published: "1994-01-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1171.PDF
---

# The argument about the arithmetic mean and the geometric mean, heuristics included

EWD 1171

The argument about the arithmetic mean and the geometric mean, heuristics included.

We consider nonempty bags of n non-negative numbers. A bag B’s “arithmetic mean” AM.B is defined as one n-th of the sum of the numbers in bag B; a bag B’s “geometric mean” GM.B is defined as the n-th root of the product of the numbers in bag B. An immediate consequence of these definitions is that if we modify the values in a bag without changing their sum (product), the arithmetic (geometric) mean of the bag remains unchanged. The theorem to be proved is

(0)         AM.B ≥ GM.B     for any B   .

Note.   The way to remember which of the two means is the greater one, is to consider a bag (with n ≥ 2) that contains exactly one zero. (End of Note.)

*         *         *

The first thing we ask ourselves is if   AM.B = GM.B   is possible.

Note   If not, the theorem could be strengthened to AM.B > GM.B; inspection if a theorem can trivially be strengthened should be a routine matter. (End of Note.)

Because

n � c
––––
n

= c       and
n
√ cn  = c     ,
the answer is affirmative: for a bag containing equal values, arithmetic and geometric mean are both equal to that value and hence equal to each other.

But this observation gives us a strong hint as how to compare AM.B with GM.B: we create a bag B� of equal elements, so that we can compare AM.B� with GM.B�, but create B� in such a fashion that AM.B can be related to AM.B� and GM.B to GM.B�. In diagram

AM.B ? GM.B

| |

| |

AM.B� ? GM.B�         .

The existence of B� enables us to replace (0), i.e. the comparison of the different means of the same bag, by (twice) the comparison of the same mean of different bags.

We try the simplest thing, and try to construct a B� such that B and B� have one of the means equal. Arbitrarily we choose the geometric mean for that purpose.

Note   In EWD1140, the arithmetic mean was chosen. (End of Note.)

The overall structure of our proof is now

AM.B

≥         {by virtue of the construction of B�}

AM.B�

=         {all elements of B� are equal to c}

GM.B�

=         {by virtue of the construction of B�}

GM.B           .

Our remaining task is to construct a program operating on a variable of type bag with B as initial and B� as final values. In order to justify AM.B ≥ AM.B�, we require

(1)         no step increases the sum.

In order to justify GM.B� = GM.B, we require

(2)         no step changes the product.

In order to assume termination, we require

(3)         each step decreases the number of elements ≠ c by at least 1.

Note that, in view of GM.B� = GM.B and GM.B� = c, c is defined as GM.B. Let c≠0.

Taking (2) and (3) into account, a first approximation of the program is

do  ¬ (all elements in the bag are equal to c) →

increase the number of elements = c

by 1 under invariance of the product

od         .

Increasing the number of elements = c is done by only changing elements ≠ c. Because the product has to remain constant, increasing one element implies decreasing another, Fortunately, because c is the geometric mean of the bag, the presence of an element different from c implies the existence of an element at c’s other side. The step therefore replaces a pair (x,y) such that c lies between x and y; it replaces the pair (x,y) by the pair (c, x�y/c). Note that, because originally c was between the elements of the pair, the minimum of the pair has been increased, the maximum of the pair has been decreased, in short: it has been a contraction, i.e. the elements of the pair have become closer to each other.

The program now becomes

do  ¬ (all elements in the bag are equal to c) →

select a pair (x,y) such that c lies between them;

replace this pair by (c, x�y/c)

od         .

Our remaining proof obligation is (1): we have to show that above step does not increase the sum. (In fact it decreases it.) We have to find a relation between

- the sum, because we have to show that the step decreases it

- the product, because the step leaves it invariant

- the difference, because its decrease distinguishes the step from its inverse.

In view of the product we are talking about a relation of the second degree, in view of the unsigned difference, we have its square, and we come up with

(4)         (x + y)� = (x − y)� + 4�x�y         ,

which is the rabbit with which EWD1140 started. It expresses that, for positive x,y, under constant product, shrinking difference and shrinking sum go hand in hand.

And this concludes our proof of (0).

Apology   I am sorry I was too late in seeing that here (in contrast to EWD1140) the case c = 0 has to be treated separately. (End of Apology.)

I am grateful to Wim H.J.Feijen for having drawn my attention to the underlying heuristics.

Austin, 19 January 1994

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
24-Dec-2011
