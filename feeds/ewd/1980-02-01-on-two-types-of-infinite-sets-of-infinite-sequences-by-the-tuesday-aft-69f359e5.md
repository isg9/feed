---
title: "On two types of infinite sets of infinite sequences (by the Tuesday Afternoon Club) (EWD730)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD730.html
published: "1980-02-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD730.PDF
---

# On two types of infinite sets of infinite sequences (by the Tuesday Afternoon Club)

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

On two types of infinite sets of infinite sequences.
by the Tuesday Afternoon Club.
In the following x will stand for the infinite sequence
(x(0), x(1), x(2), ...........)
where the x(i) are taken from a finite alphabet of at least two characters,
say {0, 1}
Let us consider the sets of sequences S1 and S2 , defined as the
solution sets of the equations P1(x) and P2(x), respectively, with

Pl(x):(E i: i ≥ 0: (A j: 0 ≤ j < i: x(j) = 0) and (A j: i ≤ j: x(j) = 1))

P2(x):(A i: i ≥ 0: x(i) = 0 or (A j: j ≥ i: x(j) = 1)) .

Because P1(x) ⇒ P2(x) , S1 is a subset of S2 , even a proper subset:
there exists one solution of P2(x) and non P1(x) , viz. the sequence with
x(1) = 0 for all i ≥ 0.
The set S2 has the property that for any non-member of S2 —i.e.
any solution of non P2(x)— non-membership can be established on account
of an initial segment of it. As a matter of fact, in this particular case
only two elements suffice for this evidence, as follows from non P2(x):

(E i: i > 0: x(i) ≠ 0 and (E j: j > i: x(j) ≠ 1)) .

The set S1 doesn’t have this property because there exists a
non-member —viz. x(i) = 0 for all i ≥ 0— with the property that any initial
segment of it is also the initial segment of a member of S1 . We call S2
“closed” and S1 “non-closed”, that is:

“The set S is closed” means “any non-member of S has an initial
segment that is not the initial segment of any member of S”.

“The set S is non-closed” means “there exists a non-member of S
such that any initial segment of it is also the initial segment of
some member of S ”.

From the above it follows

1)     that a set of sequences is either closed or non-closed,

2)     that any finite set —in particular the empty set— is closed,

3)     that the (uncountably infinite) universe of all possible sequences is
closed.
*              *
*

The notion of a closed set is of significance in connection with
non-deterministic infinite computations. Consider the set of possible output
sequences y corresponding to

“initialize; i:= 0;(1)

do true → compute; print(y(i)); i:= i + 1 od”

Here “initialize” and “compute” stand for terminating computations not
affecting the value of i . If “initialize” and “compute” stand for
deterministic computations, the set of possible output sequences consists of a single
element; if “initialize” and (in particular) “compute” are non-deterministic,
the corresponding set of possible output sequences can be infinite.

“Program (1) is a continuous machine” means “in program (1) ”initialize“
and ”compute“ are both of bounded non-determinacy”.

We can now prove the following
Theorem 1. The possible output sequences of program (1) form a closed set
or program (1) is not a continuous machine.
Proof. Let S be the set of possible output sequences of program (1).
Either S is closed —in which case Theorem 1 holds— or S is non-closed.
In the latter case, let x be a non-member of S such that each initial
segment of x is also an initial segment of some member of S . Consider
now the following program:

“initialize; i=: 0; (2)

do ”y(0),...,y(i-1) is an initial segment of x “ →

compute; print(y(i)); i:= i + 1

od; print(i)”

Non-termination of (2) is excluded because x is not a member of S
Because each initial segment of x is also the initial segment of some
member of S , the final value of i is unbounded. Hence, program (2)
is a (weakly) terminating program of unbounded non-determinacy; hence
“initialize” or “compute” is of unbounded non-determinacy, i.e. program (1)
is not a continuous machine. (End of Proof.)
Theorem 2. A set of sequences is non-closed or is the set of possible output
sequences of some continuous machine.
Proof. Let S be the set of sequences. Either set S is non-closed —in
which case Theorem 2 holds— or it is closed. In the latter case consider
program (1) with “initialize” deterministic and “compute” only constrained
by the requirement that the next y(i) leads to an initial segment of some
element of S . Because the alphabet is finite, “compute” need not be of
unbounded non-determinacy. By virtue of its construction program (1) may
then generate any member of S , and, S being closed, it cannot generate
any non-member of S . (End of Proof.)
Having identified closed sets of possible output sequences with
continuous machines, we propose to ignore for the time being non-deterministic
infinite computations to which non-closed sets correspond.

Plataanstraat 519th February 1980

5671 AL NUENENprof.dr.Edsger W.Dijkstra

The NetherlandsBurroughs Research Fellow

Last revision

2015-02-28

.
