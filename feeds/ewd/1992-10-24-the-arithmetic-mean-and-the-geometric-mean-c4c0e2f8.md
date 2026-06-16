---
title: "The arithmetic mean and the geometric mean (EWD1140)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1140.html
published: "1992-10-24T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1140.PDF
---

# The arithmetic mean and the geometric mean

EWD 1140

The arithmetic mean and the geometric mean

From the well-known algebraic identity

(x+y)� ≡ (x–y)� + 4�x�y

we conclude that, when we bring two numbers closer together without changing their sum, we increase their product. Let us call such a transformation “a contraction” of the two numbers. We furthermore observe:

(i)   when we start with two positive numbers, they remain positive when we contract them.

(ii)  let c lie between the two numbers we start with; then it is possible to contract this pair of numbers so that c occurs in —or, equivalently, “is an element of”— the resulting pair.

Consider now a bag containing n positive numbers, with n ≥ 2 (n=0 is not interesting, n=1 is trivial). Let c be their average, i.e. n.c = the sum of the n numbers in the bag.

For such a bag we observe:

(iii)  either all numbers in the bag equal c or the bag contains a pair of numbers ≠ c, such that c lies between them.

We now consider the following algorithm, operating on the bag in question

do ¬ (all numbers in the bag are equal to c) →

select a pair of numbers different from c,

such that c lies between them;

replace in the bag the pair selected by

their contraction in which c occurs

od

and we observe:

•   the step replaces in the bag a pair of numbers by a pair of numbers with the same sum as the pair replaced, hence:

the bag continues to contain n numbers and c remains their average

•   because of (i), all numbers in the bag remain positive

•   because of (iii), the selection of a pair —the first statement of the step— is possible

•   because of (ii), the second step is possible.

•   because of the increase of the product of a contracted pair, and because all other numbers in the bag are positive, the step increases the product of the n numbers in the bag.

•   the step decreases in the bag the number of numbers that differ from c by at least 1, hence the above algorithm, terminates.

So, for any initial bag B containing n positive numbers, we can construct a final bag B� of n positive numbers such that

•   sum.B = sum.B�

•   product.B ≤ product.B�

•   all elements in B� are equal to each other

The last implies that the “arithmetic mean” of B� —i.e. (sum.B�)/n— equals the “geometric mean” of B� —i.e. n
√ product.B�  —.

And now we observe —B and B� containing the same number of elements—

the geometric mean of B

≤         {product.B ≤ product.B�}

the geometric mean of B�

=         {all elements in B� are equal to each other}

the arithmetic mean of B�

=         {sum.B = sum.B�}

the arithmetic mean of B   .

The above proof was designed during the last session of the ETAC, and it is recorded because we thought it much nicer than Cauchy’s proof that J.Misra subsequentially showed us.

Lost Maples, 24 October 1992

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

transcribed by Corrado Cantelmi

revised
24-Dec-2011
