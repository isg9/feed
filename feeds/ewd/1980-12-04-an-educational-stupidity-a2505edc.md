---
title: "An educational stupidity (EWD766)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD766.html
published: "1980-12-04T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD766.PDF
---

# An educational stupidity

The following two divisibility tests are widely taught at Dutch schools.

The 9-test. To reduce a number modulo 9, add its decimal digits. E.g., with N = 15098,

15098 ≡ 8+9+0+5+1 = 23 ≡ 3+2 = 5.

It is proved by observing that, modulo 9, 10 ≡ 1, hence 10k ≡ 1. As a derivative, we are taught the 3-test: N mod 3 = 2 because 5 mod 3 = 2.

The 11-test. To reduce a number modulo 11, add its decimal digits with alternating signs, e.g.

15098 ≡ 8 - 9 + 0 - 5 + 1 = -5 ≡ 6.

It is proved by observing that, modulo 11, 10 ≡ -1, hence 10k ≡ (-1)k.

*                *

*

Here is another approach to the 11-test. Interpret the number, by pairing the digits, as written in base 100 and start with the 99-test

15098 &equiv 1+50+98 = 149 ≡ 1+49 = 50 ≡ 6,

where the last congruence is only modulo 11.
The fact that the last test manipulates only natural numbers could be viewed as a minor advantage. It is more important to see that the 11-test is related to the 99-test in exactly the same way as the 3-test is related to the 9-test. It is now immediately obvious how to reduce N modulo 111, viz. via a reduction modulo 999:

15098 ≡ 15+098 = 113 ≡ 2  ,

where the last congruence is only modulo 111. The possibility of this generalization is well-hidden in the traditional formulation of the 11-test.
Two instances of the general method are being taught as isolated tricks. I know that the mathematics involved is absolutely trivial; so much the worse that it isn't taught in all clarity!

Exercise. Prove that none of the decimal numbers 1001, 1001001, 1001001001, 1001001001001, ........ is prime.

Plataanstraat 5

4 December 1980

5671 AL NUENEN

prof.dr.Edsger W.Dijkstra

The Netherlands

BURROUGHS Research Fellow.

transcribed by Om P. Damani

last revision Thu, 8 Nov 2007
