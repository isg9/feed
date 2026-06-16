---
title: "An unavoidable case analysis (EWD1307)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1307.html
published: "2001-03-18T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1307.PDF
---

# An unavoidable case analysis

An unavoidable
case analysis

Archiving old manuscripts, I found at the end of EWD766 "An educational stupidity" of more than 20 years ago the following exercise:

"Prove that none of the decimal numbers 1001, 1001001, 1001001001, 1001001001001, ... is prime." [It is not clear why I underlined "none". EWD]

Here is a proof. Denoting by k*<string>* the concatenation of k copies of the digit string enclosed, we deal with the decimal numbers

k*001*       for k ≥ 2

- If k mod 3 = 0, the number is, according to the traditional 3-test, divisible by 3 because the sum of its decimal digits, which equals k, is divisible by 3.

- If k mod 3 ≠ 0, we see by generalising the traditional 9-test to the k*9*-test, that the number reduced modulo k*9* equals k*1*. Hence the number is divisible by k*1* (because k*9* is). (Note that this observation does not exclude primality for k=1.)

In argument (i), the crux is the validity of the 3-test, which is valid whenever base mod 3 = 1, a condition that base 10 (= ten) satisfies. The conclusion has nothing to do with the accident that the length of the repeated string happens to be 3: for k mod 3 = 0, also k*00001*  is divisible by 3.

In argument (ii) we generalized the 9-test because the problem was about decimal numbers, but for any base B (B ≥ 2) there is a (B−1)-test, and k*1* divides k*(B−1)*. The conclusion that the number reduced modulo k*(B−1)* yields k*1*, however, depends on the fact that k has no factor in common with the length of the iterated string. Argument (ii) is base independent: interpreting *k001* and *k1* in binary, we find for instance that for k=4 and k=5, 585 is divisible by 15 and 4681 by 31.

Argument (i) relies on a relation between k and the base of the number system, (ii) on a relation between k and the length of the iterated string.

Austin, 18 March 2001

prof.dr. Edsger W. Dijkstra

Department of Computer Science

The University of Texas at Austin

Austin, TX 78712-1188, USA

Transcriber: Kevin Hely.

Last revised on Sat, 24 Nov 2007.
