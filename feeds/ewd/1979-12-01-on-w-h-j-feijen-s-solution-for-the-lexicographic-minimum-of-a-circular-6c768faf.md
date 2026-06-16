---
title: "On W.H.J.Feijen’s solution for the lexicographic minimum of a circular list (EWD723)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD723.html
published: "1979-12-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD723.PDF
---

# On W.H.J.Feijen’s solution for the lexicographic minimum of a circular list

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

On W.H.J. Feijen's solution for the lexicographic minimum of a circular list.
For a given, non-empty —i.e. with N ≥ 1— list
(C[0], C[1], ..., C[N-1])
we consider the N associated sequences Ci

Ci = (C[i], C[i+1], ..., C[i+N-1])

in which all subscripts are reduced modulo N.
We are asked to design a program determining
an integer value k satisfying the relation

R:0 ≤ k < N and (∀i: 0 ≤ i < N: Ci GE Ck) ,

in which "GE" should be read as "lexicographically greater or equal than".
*              *
*

The lexicographic minimum CC is defined by the following two relations.

(∀i: 0 ≤ i < N: Ci GE CC)(0)

(∃i: 0 ≤ i < N: Ci EQ CC)(1)

From the transitivity of the lexicographic ordering we conclude

(∀p, q:: Cp GE Cq or Cq GT CC)(3)

Note. When, in a quantification, the range is obvious,
we allow ourselves its omission. Hence the "::". (End of note)
*              *
*

Let P(k,n) be defined as:

P:
0 ≤ k < n and (∀i; 0 ≤ i < n: Ci GT CC or i = k)(4)

We observe immediately

(P and n ≥ N) ⇒ R(5)

(∀i: 0 ≤ i < h: Ci GT CC) ⇒ P(h, h+1)(6)

*              *
*

Let Q(k, n, l) be defined as

Q:
0 ≤ k < n ∧ (∀i: 0 ≤ i < l: C[k+i] = C[n+i])(7)

Because the lexicographic order of the two sequences with equal leading elements
does not change when those leading elements are removed from them, we conclude

Q(k,n,l) ⇒ (Qa or Qb or Qc)(8)

with the mutually exclusive terms

Qa:
(∀i: 0 ≤ i < N: Ck+i = Cn+i (8a)

Qb:
(∀i: 0 ≤ i < l: Ck+i LT Cn+i ∧ l ≥ 0(8b)

Qc:
(∀i: 0 ≤ i < N: Ck+i GT Cn+i ∧ l ≥ 0(8c)

We furthermore observe that, for l=0, the
last term of Q(k, n, l) is vacuously true.
*              *
*

We observe that P&ans;Q can be initialized by

k, n, l:= 0, 1, 0

and is left invariant by the following three guarded commands.

1)C[k + l] = C[n + l] → l:= l + 1

This follows immediatly from (4) —l does not occur in P— and (7)

2)C[k + l] < C[n + l] → {P∧Qb} n, l:= n + l + 1, 0

Q and the guard imply together Qb, and note that Qb implies

(∀i: n ≤ i < n+l+1: CI GT CC)

Hence, on account of (4) the invariance of P is guaranteed.
The assignment l:= 0 ensures the invariance of Q

3)C[k + l] > C[n + l] → {P∧Qc} n, l:= n + l + 1, 0

h:= max(n, k+l+1);

k, n, l:= h, h+1, 0

Q and the guard imply together Qc, and note that Qc implies

(∀i: k ≤ i < n+l+1: Ci GT CC) ∧ l ≥ 0 .

With h = max(n, k+l+1) we conclude from the last result and P

(∀i: 0 ≤ i < h: Ci GT CC)

and, thanks to (6), the invariance of P is guaranteed.
The assignment l:= 0 ensures the invariance of Q .
*              *
*

Finally we show that

(P ∧ Q ∧ k + l + 1 ≥ N) ⇒ R(9)

1)P ∧ Qa ∧ k + l + 1 ≥ N

From Qa we conclude that the sequence is
periodic, and that the period divides n - k; hence

(∃i: k ≤ i < n: Ci EQ CC)

Together with —what follows from P—

(∀i: k < i < n: Ci GT CC)

we conclude Ck EQ CC, i.e. R .

2)P ∧ Qb ∧ k+l+1 ≥ N

From Qb we conclude, as before,

(∀i: n ≤ i < n+l+1: Ci GT CC);

because n > k, n+l+1 > k+l+1; because k+l+1 ≥ N
we conclude, together with P

(∀i: 0 ≤ i < N: Ci GT CC or i = k), i.e. R

3)P ∧ Qc ∧ k+l+1 ≥ N

From Qc we conclude, as before,

(∀i: k ≤ i < k+l+1: Ci GT CC);

because k+l+1 ≥ N, we conclude, together with P

(∀i: 0 ≤ i < N: Ci GT CC);

this together with (1) implies false, and false ⇒ R.
(End of Proof of (9).)
Using (5) and (9), we see that the following program establishes R

k, n, l:= 0, 1, 0

do n < N and k + l + 1 < N →

if C[k + l] = C[n + l] → l:= l + 1 {P ∧ Q}

▯ C[k + l] < C[n + l] → n, l:= n + l + 1, 0 {P ∧ Q}

▯ C[k + l] > C[n + l] → h:= max(n, k + l + 1);

k, n, l:= h, h + 1, 0 {P ∧ Q}

fi {P ∧ Q}

od {R}

Termination. The function t = k + n + l increases by at
least one at each iteration. Before the first iteration t = 1,
before the last one t ≤ 2N - 3, hence 2N - 3 is an
upper bound for the number of comparisons.

*              *
*

The above is —or will be— described much more
beautifully by W.H.J. Feyen, including all the heuristics and acknowledgements.
I wrote the above as part of my personal correspondence, for the sake
of quick dissemination: I owe everything of the above to ir. W.H.J. Feijen.

29th December 1979

Plataanstraat 5

5671 AL Nuenenprof.dr.Edsger W.Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

2015-04-11

.
