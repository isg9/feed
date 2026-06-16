---
title: "On weak and strong termination (EWD673)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD673.html
published: "1979-05-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD673.PDF
---

# On weak and strong termination

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 673 On weak and strong termination

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 355�357 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

On weak and strong termination.
In the literature we find two concepts of “termination”; we shall
call them “weak termination” and “strong termination” respectively, equivalent
within the realm of continuous functions, but different in the presence of
unbounded nondeterminacy. It will be shown that in the realm of continuous
functions the generality of (infinite) well-founded sets is of no essential
use for proofs of termination, as partially ordered finite sets will do just
as nicely.
*              *
*

In a proof of weak termination we demonstrate the impossibility that
a computation will continue “forever”, although an upper bound on the “time”
it will take need not exist; in a proof of strong termination we demonstrate
that the computation will have terminated within a certain amount of “time”.
For proofs of strong termination the conceptually simplest tool is
the so-called “variant function”, an integer-valued function of the state
which is bounded from below ( ≥ 0, say), and decreased by at least 1 at
each “step” of the computation.
For proofs of weak termination Floyd [1967] has suggested to replace,
as range of the variant function, the natural numbers by the elements of a
so-called “well-founded set”. A well-founded set is a set on which a
(partial) ordering has been defined such that no element is the first of an
infinite decreasing sequence of elements from the set. A well-known example
of a well-founded set is the one consisting of the pairs (x,y) of natural
numbers with the ordering defined as

(x',y') < (x,y)   =
def
x' < x or (x'= x and y' < y) .

This well-founded set would be the proper vehicle for proving the weak
termination of — X and Y being natural constants—
S:
x, y := X, Y;

do x > 0 → x, y := x-1, any natural number

▯ s > 0 → y:= y - 1

od

where “any natural number” denotes a function of unbounded nondeterminacy,

i.e. such that

wp(“y:= any natural number”, y ≥ 0) = T and

wp(“y:= any natural number”, y ≤k) = F for all k .

Note that in general program S does not enjoy the property of strong
termination, because for X > 0 no upper bound for y can be given.
The well-founded set of the pairs (x,y) used above nicely illustrates
the way in which well-founded sets are a true generalization of the natural
numbers. Each natural number n is the first element of only finite
decreasing sequences, but only of a finite number of them — 2n , to be
precise— that, therefore, have a maximum length — n+1 , to be precise— .
In the more general well-founded set we considered, each element (x,y)
with x ≥ 1 is the first element of only finite decreasing sequences, but
of infinitely many of them, whose lengths have no maximum. Our example
also suggests that the generality the well-founded sets offer over and
above the natural numbers is the last thing we need.
With program S we showed how, under assumption of the availability
of the function “any natural number” of unbounded nondeterminacy, we could
implement a weakly terminating program that was not strongly terminating.
On the other hand it is quite easy to derive from any weakly terminating
program that does not terminate strongly a computation of “any natural
number”: just add to it a count of the number of “steps” executed.
Therefore the availability of the function “any natural number” of unbounded
nondeterminacy is equivalent to the existence of programs that terminate
weakly, but not strongly. Furthermore it is known —see, for instance,
Dijkstra [1976], Chapter 9 — that unbounded nondeterminacy is
incompatible with the constraint of continuity.
Several conclusions present themselves:

1)     Within the realm of continuous functions, where nondeterminacy is
bounded, weak termination and strong termination are equivalent.

2)     We only need the greater generality of the well-founded sets over
and above the natural numbers, when we decide to leave the realm of the
continuous functions. As long as there is very little incentive to do so,

that greater generality of (infinite) well-founded sets is of no essential
use, and (partially) ordered finite sets will do just as nicely. (As a
partial order on a finite set can always be embedded in a total order, the
prevalence of the use of the range of natural numbers —the first K , for
some sufficiently large K , to be precise— becomes now fully understandable.)
Dijkstra, Edsger W. [1976] “A Discipline of Programming”, Prentice-Hall,
Englewood Cliffs, NJ, U.S.A.

Floyd, R.W. [1967] “Assigning Meanings to Programs”. Proc. Symp. in Applied
Mathematics, vol. 19 (J.T.Schwartz, ed.), American Mathematical Society,
Providence, RI, U.5.A.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENENBURROUGHS Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-02-05

.
