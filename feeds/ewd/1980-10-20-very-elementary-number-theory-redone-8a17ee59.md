---
title: "Very elementary number theory redone (EWD755)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD755.html
published: "1980-10-20T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD755.PDF
---

# Very elementary number theory redone

The natural numbers are 0, 1, 2 and so on; for further details, see Peano.
The positive integers are natural numbers ≥ 1, i.e. 1, 2, 3 and so on.
The multiples are natural numbers ≥ 2, i.e. 2, 3, 4 and so on. The product of two multiples is larger than each of those two, in formula

(A x, y : x ≥ 2, y ≥ 2 : x.y > x ∧ x.y > y ) (1)

Multiples that are the product of two multiples are called composite, multiples that are not composite are called prime, in formula

prime m = m ≥ 2 ∧ (A x, y : x ≥ 2, y ≥ 2 : x.y ≠ m) (2)

comp m = m ≥ 2 ∧ (E x, y : x ≥ 2, y ≥ 2 : x.y = m) (3)

In view of (1), (3) can be rewritten

comp m = (E x, y : 2 ≤ x < m, 2 ≤ y < m : x.y = m) (3′)

From (3′) it is clear that it is decidable whether a specific m is composite.
Note that we have ¬(prime 1) ∧ ¬ (comp 1) .
With PF —for Prime Factorization— defined by

PF n =n is the product of (0 or more) multiples, all of which are prime (the empty product being defined as 1)

we can state
Theorem 0. (A n : n ≥ 1 : PF n)
Proof. We shall prove Theorem 0 by mathematical induction, i.e. we shall prove

(A n : n ≥ 1 : (PF n) ∨ (E i : 1 ≤ i ≤ n : ¬(PF i))) (4)

Because (A n : n ≥ 1 : n = 1 ∨ (prime n) ∨ (comp n))
and the product of one number equals that number, we deduce from the definition of PF

(A n : n ≥ 1 : (comp n) ∨ (PF n)) (5)

Furthermore we deduce from the definition of PF
(A x, y : x ≥ 1, y ≥ 1 : ¬(PF x) ∨ ¬(PF y ) ∨ PF(x.y ))
from which we deduce with (3')

(A n : n ≥ 1 : ¬(comp n) ∨ (PF n) ∨ (E i : 1 ≤ i ≤ n: ¬ (PF i ))) (6)

From (5) and (6), relation 4 follows by the standard inference rule. (End of Proof of
Theorem 0).
Theorem 1. (A p : prime p : (E q : q > p : prime q ))
(This is Euclid's famous theorem that no prime is the largest one.)
Proof. Consider for arbitrary prime p the value Q defined by
Q = 1 + the product of all primes ≤ p.
Theorem 0 allows us to conclude that Q is the product
of a bag of primes, and, because Q > 1, that bag is
not empty. By virtue of Q’s construction, such a bag
contains no prime ≤ p. Hence it contains at least one prime > p, hence at least one
prime > p exists. (End of proof of Theorem 1.)
With UPF —for Unique Prime Factorization— defined by
UPF n = The bag of prime multiples whose product equals n is unique
we can formulate
Theorem 2.
(A p, x, y : p ≥ 1, x ≥ 1, y ≥ 1 :
¬(prime p) ∨ ¬(p|(x.y ))  ∨  p|x  ∨  p|y  ∨  ¬(UPF(x.y )))
(Here “a|b ” should be read as “a divides b”.)
Proof. When the first two terms are false, x.y is the product
of a bag of primes containing p ; when the next two terms are false, x.y is also the
product of a bag of primes not containing p. Those two bags being different,
we deduce ¬(UPF(x.y )).
(End of proof of Theorem 2.)
We are now ready to state and prove
Theorem 3. (A n : n ≥ 1 : UPF n)
Proof. Theorem 3 is proved by mathematical induction, i.e. by proving

(A n : n ≥ 1 : (UPF n) ∨ (E i : 1 ≤ i < n : ¬(UPF i ))) (7)

Inspired by our proof of Theorem 0, we observe that (7) follows from (8) and (9), given by

(A n : n ≥ 1 : (comp n) ∨ (UPF n)) (8)

(A n : n ≥ 1 : ¬(comp n) ∨ (UPF n) ∨ (E i : 1 ≤ i < n : ¬(UPF i ))) (9)

Of these two, (8) follows immediately from (2), (3) and the definition of UPF. Assertion (9) requires,
however, a more elaborate argument.
Consider a value of n, m say, such that the first and last terms of (9) are false, i.e. such that

(comp m) ∧ (A i : 1 ≤ i < m : UPF i ) (10)

holds. Because m is composite, it can be written as

m = p.P = q.Q with (11)

1)prime p

2)p ≤ q

3)P standing for the product of a non empty bag of primes all of which are ≥ p

4)Q standing for the product of a bag of primes all of which are ≥ q.

We obviously have 2 ≤ P < m, 1 ≤ Q < m, and P ≥ Q.
The conclusion UPF m, required to prove (9), can be drawn from

p = q ∨ (comp p) (12)

as follows. When q is the smallest value in a bag of primes, comp q is false and from (12) we deduce
p = q. From (11) we deduce P = Q, and from (10) we deduce that P and Q stand for the same
bag of primes. Since p = q, p.P and q.Q then also stand for the same bag of primes,
and UPF m has been established
Establishing (12) is now our only remaining obligation. It is trivial in the case p = q. In
the case p < q we consider value the M defined by

M = (q−p) . Q

Because 1 ≤ M < m, we conclude from (10) UPF m. Because M = q.Q −p.Q = p.P−p.Q = p.(P−q), we conclude p|M. Since
prime P, we now conclude from theorem 2

p | (q−p) ∨ p|Q(13)

Since 1 ≤ Q < m, we conclude from (10) UPF Q, i.e. all the primes in the unique bag of
primes whose product equals Q are ≥ q, hence > p. The bag being unique, we
conclude ¬(P|Q ). In combination with (13) we conclude p|(q−p); since we were considering
the case p < q, we can conclude that q is composite. Having thus established
relation (12), we have fulfilled our last proof obligation. (End of Proof of Theorem 3.)
Corollary of theorems 2 and 3.

(A p, x, y : p ≥ 1, x ≥ 1, y ≥ 1 : ¬(prime p)  ∨
¬(p|(x.y ))  ∨  p|x  ∨  p|y ) .
The above was written in reaction to the presentation in [1] of essentially the same proof
of Theorem 3. I found that presentation contorted and notationally clumsy. They write (for (11))

m = p1 p2 ... pr = q1 q2 ... qs

Note the dots and the need to introduce the subscripts r and s. A little later, after having
excluded p1 equals q1:
“Suppose p1 q1. (If q1 < p1,
we simply interchange
the letters p and q in what follows (sic).)”. A third of a page later they conclude that
“the prime decomposition of m′ must be unique, aside from the order of the factors”.
Theorem 2 isn't mentioned and
when needed in the proof of Theorem 3 they borrow the result from a corollary of theorem
3! (Here their argument is almost cyclic.) And it is full of reductiones ad absurdum.
[1] Courant, Richard & Robbins, Herbert,
What is mathematics? Oxford University Press paperback,
1978.

Plantaanstraat 520 October 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
