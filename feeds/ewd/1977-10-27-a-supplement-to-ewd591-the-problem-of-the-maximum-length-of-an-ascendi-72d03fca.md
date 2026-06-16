---
title: "A supplement to EWD591 “The problem of the maximum length of an ascending subsequence” (EWD649)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD649.html
published: "1977-10-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD649.PDF
---

# A supplement to EWD591 “The problem of the maximum length of an ascending subsequence”

A supplement on EWD591 “The problem of the maximum length of an ascending subsequence.”

In the algorithm described in EWD591 occur the array elements   m(1)   through   m(k)   where   k = the maximum length of an ascending subsequence taken from A(1) through A(n). In the computational step that increases   n   by   1  , either   k   is increased by   1   and the m-sequence is extended by a value ( = the new A(n), to be precise), or for some value   j   (1 ≤ j ≤ k) m(j), that was larger than the new   A(n)   is made equal to the new   A(n)  . In short: after the adjustment the new   A(n)   occurs in the m-sequence and existing m-values never increase.

Suppose that in parallel we determine   h = the maximum length of a descending subsequence taken from   A(1)   through   A(n)  . That computation would comprise a corresponding array   p(1)   through p(h), that after each adjustment would contain the new   A(n)   and whose elements never decrease. If, for a given   i   (1 ≤ i ≤ h)   and   j   (1 ≤ j ≤ k)   we have   m(j) ≤ p(i)   we call this “an inversion”; because   m(j)   never increases and   p(i)   never dercreases, an inversion, once introduced, remains in existence.

If we mentally extend the m-sequence at the high end with values = + infinity, and extend the p-sequence at the high end with values = – infinity, each step effectively decreases an m-value by making it equal to the new A(n) and effectively increases a p-value by making it equal to the new A(n). Hence each step increases the total number of inversions by at least one, and we conclude that the total number of inversions ≥ the length n of the sequence. Furthermore h*k ≥ the total number of inversions, hence

h * k ≥ n .

From this we deduce that for   n ≥ N2 + 1,   we have   h > N or k > N,   and that was the theorem we wanted to prove:   a sequence of length greater than   N2   has an ascending or descending subsequence of length greater than   N   .

Plataanstraat  5 prof. dr. Edsger W. Dijkstra

5671  AL  Nuenen Burroughs Research Fellow

The Netherlands

transcribed by Swarup Sahoo
last revised Fri, 5 Aug 2011
