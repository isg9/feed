---
title: "On subgoal induction (EWD576)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD576.html
published: "1976-07-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD576.PDF
---

# On subgoal induction

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 576 On subgoal induction

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 223�224 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

On subgoal induction.
In [1] I encountered “subgoal induction” as a technique for proving partial
correctness. It was applied to a program S that I would write down as

S : x:= f(x0);

do B(x) → x:= g(x) od;

x:= h(x) .

In order to prove

{P(x0)} S {R(x0, x)}(1)

—i.e. if P(x0) holds and the execution of S terminates properly, then in
the final state R(x0, x) will hold— “subgoal induction” is used. The technique
consists of finding a relation Q(x, z) satisfying

(A x: (non B(x)) ⇒ Q(x, z))(2)

(A x, z: (Q(g(x), z) and B(x) ⇒ Q(x, z))(3)

(A x, z: (P(x) and Q(f(x), z)) ⇒ R(x, z))(4)

and it was stated that the existence of a relation 0 satisfying (2), (3) and
(4) proves (1).
My general inclination when I encounter such formulae —particularly when
I encounter them in a report that is really dealing with something else— is to
skim them, assuming that they are no more than variations on an old theme.
Formula (3), however, attracted my attention, because, if P’(x) is the invariant
relation for the repetitive construct, we have to prove —see [2]—

(P’(x) and B(x) ⇒ P’(g(x))(5)

and, if we compare (5) with (3), we see that the substitution of g(x) for x
occurs at the other side of the implication! This was reason enough to
investigate subgoal induction a little bit more closely.
In terms of a relation Q satisfying (2), (3), and (4), we can take as
our invariant relation

P’(x): (A z: Q(x, z) ⇒ Q(f(x0), z))(6)

a relation which is clearly established by “x:= f(x0)” , the first statement of
S . To prove (S) we have to prove

((A z: Q(x, z) ⇒ Q(f(x0), z)) and B(x)) ⇒
((A z: Q(g(x), z) ⇒ Q(f(x0), z))(7)

For those values of x , such that B(x) is false, the implication (7) is
vacuously true, for those values of x , such that B(x) is true, (3) tells us that
Q(g(x), z) is a stronger condition on 2 than Q(x, z) , so that whatever is
implied by the latter is certainly implied by the former. Hence (7) and thus (5)
follows from (3).
Finally we have to prove that

(P’(x) and non B(x)) ⇒ wp(“x:= h(x)”. R(x0, x))(8)

Thanks to (2) and (6), the left-hand side of (8) reduces to

(A z: Q(x, z) ⇒ Q(f(x0), z)) and Q(x, h(x))

from which we conclude —applying the quantified implication for z = h(x)—
the truth of

Q(f(x0), h(x)) .

Because the initial value x0 satisfies P(x0) , we conclude —applying (4)
with x = x0 and z = h(x)— the truth of

R(x0, h(x))

but thanks to the axiom of assignment this is identical to the right-hand side
of (8). Hence (8) frollows from (2), (4), and (6).
Thus we have established that —as was to be expected— subgoal induction
is indeed the next variation on an old theme.
The analysis described above was carried through together with C.S. Scholten.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

NL-4565 NUENENBurroughs Research Fellow

The Netherlands

[1] Is “sometime” sometimes better than “always”? Intermittent assertions in
proving program correctness, by Zohar Manna and Richard Waldinger, STAN-C5-76-558

[2] Guarded Commands, Nondeterminacy and Formal Derivation of Programs, by
Edsger W.Dijkstra, Comm.ACM 13, 3 (Aug.1975) 453 - 457.

Transcribed by Martin P.M. van der Burgt

Last revision

2015-01-23

.
