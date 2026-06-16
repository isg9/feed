---
title: "Factorizing the factorial (EWD1041a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1041a.html
published: "1989-02-12T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1041a.PDF
---

# Factorizing the factorial

Let n be a natural number; let p be a prime;

m = the exponent of p in the prime factorization of n! ;

s = the sum of the (p-ary) digits of n’s representation in base p.

Theorem      m = ( n - s ) / ( p - 1 )

Because of the usual recursive definition of n!, an inductive proof over n seems indicated. Since for n = 0, m = 0 ∧ s = 0, the theorem holds for n = 0.

Consider now the transition from n to n + 1; let k be the exponent of p in the prime factorization of n + 1.

Because ( n + 1 )! = ( n + 1 ) ∙ n! and p is prime

(0)      Δ m = k .

Because ( n + 1 ) - n = 1

(1)      Δ n = 1 .

Because the p-ary representation of ( n + 1 ) ends on exactly k zeros

(2)      Δ s = 1 - k∙( p - 1 )

Combining (1) and (2) yields

(3)      Δ (( n - s ) / ( p - 1 )) = k .

Combining (0) and (3) yields the induction step, and thus the proof is completed.

Edsger W.Dijkstra

transcribed by Jose Alejandro Ruiz Delgado
revised Sun, 31 Aug 2008
