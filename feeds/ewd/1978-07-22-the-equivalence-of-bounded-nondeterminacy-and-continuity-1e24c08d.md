---
title: "The equivalence of bounded nondeterminacy and continuity (EWD675)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD675.html
published: "1978-07-22T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD675.PDF
---

# The equivalence of bounded nondeterminacy and continuity

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 675 The equivalence of bounded nondeterminacy and continuity

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 358�359 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

The equivalence of bounded nondeterminacy and continuity.
Unbounded nondeterminacy is presented by the function “any natural
number” such that

wp("x:= any natural number", 0 ≤ x) = T

wp("x:= any natural number", x ≤ k) = F for all k .

Program S is continuous —see Chapter 9 of “A Discipline of
Programming”, where this property is called Property 5— means that for any
infinite sequence of predicates C0 , C1 , C2 , ... such that

for r ≥ 0     Cr ≠ Cr+1     for all states

we have for all states

wp(S, (E r: r ≥ 0: cr)) = (E s: s ≥ 0: wp(S, cs))(1)

and in the same chapter I have shown that all programs that could be
written in my programming language fragment —with finite (1) guarded
command sets— are continuous.
It is further shown that the program “x:= any natural number” is
not continuous —and, hence, cannot be written in that programming language
fragment— . For the sake of completeness, we repeat the proof. Assume
the program S: “x2: any natural number” to be continuous. We then have:

T = wp(S, 0 ≤ x)

= wp(S, (E r: r ≥ 0: 0 ≤x ≤ r))

= (E s: s ≥ 0: wp(S, 0 ≤ x ≤ s))

= (E s: s ≥ 0: F) = F

a contradiction that leads to the conclusion that “x:= any natural number”
cannot be continuous, i.e. that continuity implies bounded nondeterminacy.
In the sequel of this note we shall show that the inverse holds as well,
viz. that the existence of a noncontinuous program implies the inclusion of
unbounded nondeterminacy. (The following argument was suggested to me by
C.S.Scholten almost instantaneously when I had posed the problem.)
Assume the existence of a program S and an infinite sequence of

predicates C such that Cr ⇒ Cr+1, such that (1) does not hold. Because
in (1) the right-hand side implies the left-hand side trivially, this means
that we assume

wp(S, (E r: r ≥ 0: cr))and non (E s: s ≥ 0: wp(S, cs)) =

wp(S, (E r: r ≥ 0: cr))and (A s: s ≥ 0: non wp(S, cs))(2)

to be different from F .
Consider now the program

S; x:= (MIN: k: Ck)

started in an initial state satisfying (2). Because the initial state
satisfies wp(S, (E r: r ≥ 0: Cr)) , this program terminates and is
guaranteed to establish 0 ≤ x . On the other hand, the assumption that
for some K it is certain to establish x ≤ K means that S is certain
to establish CK , a conclusion that is incompatible with the second term
of (2). Hence its nondeterminacy is unbounded. (The fact that our program
of unbounded nondeterminacy is not a total program, but only defined for
initial states satisfying (2) is here not relevant: the essential thing is
that (2) differs from F , i.e. that the set of states satisfying (2) is
not empty.)
Here we have established the equivalence of continuity and the
boundedness of nondeterminacy. In EWD673 we have established the equivalence between
the boundedness of nondeterminacy and the equality between weak and strong
termination. Hence the three criteria

1)     continuity or not

2)     nondeterminacy bounded or not

3)     weak and strong termination equivalent or not
are three different aspects of the same dichotomy. All this is very
satisfying. (The arguments are so simple that, presumable, this is already
known. But it was new for me, and I like the arguments.)

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENENBURROUGHS Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-02-05

.
