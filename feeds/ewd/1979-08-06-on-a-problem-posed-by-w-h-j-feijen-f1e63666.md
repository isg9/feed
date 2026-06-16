---
title: "On a problem posed by W.H.J.Feijen (EWD713)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD713.html
published: "1979-08-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD713.PDF
---

# On a problem posed by W.H.J.Feijen

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

On a problem posed by W.H.J.Feijen.
Given, for M ≥ 0 and N ≥ 0 , the strictly increasing sequences

F(0) < F(1) < ... < F(M-1)

G(0) < G(1) < ... < G(N-1)

and the desired final state

R:Q(M, N)

where the predicate Q has been given by

Q(m, n):(E (i, j): 0 ≤ i <m, 0 ≤ j < n: F(i) = G(j)) = k

(the “N” to be read as: “the number of distinct pairs (i, j) with i and
j in the ranges so and so, such that such and such”.) W.H.J.Feijen posed
the problem of giving a formal derivation, heuristics included, of an efficient
program establishing R .
The standard way of solving this problem would be “Replacing constants
by variables”, i.e. would be to introduce two local variables —which we
shall again denote by my and n — , their role being given by the invariant

P:Q(m, n) and 0 ≤ m ≤ M and 0 ≤ n ≤ N .

easily initialized by

“m, n, k := 0, 0, 0”

and enjoying the obvious property

(P and m = M and n = N) ⇒ R .(1)

(It could be remarked that also initializations such as “m, n, k := 0, N, 0”
or m, n, k := M, 0, 0“ would have done the job, but symmetry should never
be destroyed lightly.)
Appealing to (1) implies a final state with (m, n) = (M, N) ; the
crucial observation is that the transition to it from the initial state
(m, n) = (0, 0) can be made via so many different paths (increasing m or/
and n by 1 at a time) that we should raise the question whether we can

exploit this freedom by strengthening P into P’ and by weakening the
condition for termination. (The weaker the condition for termination, the
stronger the guards and the sooner we may expect the repetition to stop.)
We can weaken the condition for termination by not requiring that both the
local variables m and n have reached their maximum value, i.e. we find
ourselves looking for a P’ , stronger than P , and such that

(P’ and (m = M or and n = N)) ⇒R(2)

Investigating the case m = M , we find ourselves facing the question:
what more should be known besides P such that we may conclude R ? In
view of the monotonicity of the sequences F and G it would suffice to
know that F(m-1) < G(n) because

(Q(M, n) and F(M-1) < G(n)) ⇒ R .

Because we are only interested in the inequality F(M-1) < G(n) —itself
to strong to be included in the invariant— the inclusion of F(m-1) < G(n)
would suffice. Hence we find ourselves considering the invariant (symmetry!)

P’:P and F(m-1) < G(n) and G(n-1) < F(m) ,

an invariant that admits the same initialization if we define formally
F(-1) = G(-1) : minus infinity .
The standard routine of computing wp(“m:= m + 1”, P’), wp(“n:= n + 1, P”),
and wp(“m, n, k := m + 1, n + 1, k + 1”, P“) and simplifying by omitting
implied terms from the conjunctions gives the guards; the following program
results:

m, n, k := 0, 0, 0; {P'}

do m ≠ M and n ≠ N →

if F(m) < G(n) → m:= m + 1 {P'}

▯ G(n) < F(m) → n:= n + 1 {P'}

▯ F(m) = G(n) → m, n, k := m + 1, n + 1, k + 1 {P'}

fi {P'}

od

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENEN Burroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-02-19

.
