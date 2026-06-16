---
title: "On well-shaped mathematical arguments (EWD729)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD729.html
published: "1980-02-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD729.PDF
---

# On well-shaped mathematical arguments

EWD 729

On well-shaped mathematical arguments.

The following is not written with the intention of making myself popular.
On the contrary, I am afraid that I can think of sizeable subcultures
—mathematicians, logicians, philosophers, psychologists, and pedagogues,
to name just a few— whose primary reaction will be to be annoyed rather
than amused or enlightened.

This text is written in the firm conviction that there is a profound
difference between the notions "convenient" and "conventional". Its
actual writing has been prompted by the observation that many people
have great difficulty in making the distinction. Convenience should be
an objective notion, independent of our personal habits and educational
pasts, but many people, while intellectually agreeing that performing
computations in decimal (Arabic) notation is more convenient in such an
objective sense than doing it in Roman numerals, will make a moment later
their notion of convenience again dependent on "what you are used to" (and
this appeal to personal taste will be defended with adjectives such as
"natural" and "intuitive"). But our tastes are formed by our habits, and
our mathematical tastes are no exception: they are formed by our
mathematical habits. Hence my desire to subject the latter to a closer
scrutiny.

*                      *

*

When I was a pupil at the Gymnasium Erasmianum in Rotterdam, Euclidean
Geometry was a major component of the curriculum. As far as I was
concerned this needed no further justification, for I liked the topic.
It was, however, explicitly justified on the grounds that Euclidean
Geometry was an excellent topic to be used to teach us how to think
logically. In a way it was, but were all the accompanying habits wholesome? I don't think so. We learned how to prove theorems, and that was good. We also learned the profound difference between an unproved conjecture and a proven theorem, and that was good too. But in retrospect I have equally profound objections to the way in which the theorems used to be formulated: they used to follow the pattern "if P, then Q" or, equivalently, "given P, prove Q". And this has strange consequences.

In one of the first chapters of a recent (!) book on geometry I found
the following two theorems (I quote by heart):

Theorem 1. If, in a triangle, two angles are equal, then these two
angles have bisectors of equal length.

Theorem 2. If, in a triangle, two angles have bisectors of equal
length, then these two angles are equal.

The authors need a full page for the explanation why the second theorem is
much harder to prove than the first one. While the first theorem can be
proved "directly", the second theorem can be better proved "indirectly",
i.e. it is easier to prove Theorem 2' "instead":

Theorem 2'. If, in a triangle, two angles are different, then these
two angles have bisectors of different length.
After having proved Theorem 2' —the so called "counterpositive"— the proof
of the original theorem follows immediately. Suppose that Theorem 2 were
false; then there would exist a triangle with two angles with bisectors of
equal length that would be two different angles; Theorem 2' then tells us
that their bisectors would be of different length. Because two bisectors
cannot be simultaneously of equal and of different length, we have reached
a contradiction. Hence our assumption that Theorem 2 was false is untenable,
and, hence Theorem 2 is true.

What a terrible smokescreen! The authors failed to stress that Theorem 2 and
Theorem 2' are exactly the same theorem, only formulated differently.
With Theorem 2 of the form "if P, then Q", Theorem 2' is of the form "if
(non Q), then (non P)", or, using the implication sign, they
are of the forms

"P => Q" and "(non Q) => (non P)"

respectively. Remembering that the implication A => B is only an asymmetric way of writing the disjunction (non A) or B, we can equate the two formulations to

"(non P) or Q" and "Q or (non P)"

respectively; the disjunction being symmetric, the two expressions are clearly equivalent.

The whole folklore about "direct proofs" and "indirect proofs" and the medieval
relic of "the counterpositive" could have been disposed of by formulating
the theorems as disjunctions to start with!

Theorem I. Two angles of a triangle are different or they have
bisectors of equal length.

Theorem II. Two angles of a triangle are equal or they have bisectors
of different length.

It is my contention that it is (objectively!) more convenient to represent the
symmetric relation A or B by "A or B" (or by "B or A")
than by one of the asymmetric expression "(non A) => B" and
"(non B) => A". Here one of my beliefs —or if you prefer: one of my
considered opinions— surfaces: an asymmetric notation for a symmetric relation is
a pain in the neck, and the pain is misleading.

The apparent "greater difficulty" of Theorem 2 is no more than an artefact,
created by an inadequate notation. How utterly misleading that notation is I
learned when I confronted an elderly mathematician (for whom I have nothing
but the greatest respect) with Theorem 1 and Theorem I. It was for him not
easy at all to "admit" their equivalence, because emotionally he processed the
two quite differently: Theorem 1 was for him a theorem concerned with
triangles with equal angles, while Theorem I was about any triangle.
Though logically equivalent, they managed to "mean" different things for him.

When we have to demonstrate the truth of theorems of the form A or B or
A or B or C, we have to show that under any circumstances at least
one of the (two or three) terms is true. The general pattern of such proofs is
the exhaustive investigation of the situations that are possible when all the
terms but one are false; when we can show that in all these situations the
remaining term is true, we have proved the theorem. With a theorem of the
form A or B we have two different proof strategies, with a theorem
of the form A or B or C we have the choice the choice between
three different strategies. The point is that the choice is absolutely free
and should be made consciously. We should always ponder about the question
which set of situations to be investigated is the most manageable. (In the
case of the theorems I and II, they are the situations in which something about
the angles has been given.)

It is, for instance, exactly this freedom that should be wisely exploited
in many of the problems posed in the Mathematical Olympics. Let me give you
one example —again I quote by heart—

"Nine mathematicians meet at an international congress. When it is given
that of any three of them, at least two can communicate in a common
language, and no mathematician masters more than three languages, show
that there exists a language common to at least three of them."

Well, this is a theorem of the form "A or B or C" with

A:   there exists a triple of the mathematicians that is incommunicado (i.e. such that no two of the triple have a language in common)

B:   there exists a mathematician mastering more than three languages

C:   there exists a language mastered by at least three mathematicians.

The asymmetric way in which the problem has been stated suggests to investigate
the "given" situations, characterized by "(non A) and
(non B)". Condition A being the most awkward one —concerning, as it does,
pairs from triples— we may expect the set of possible situations satisfying
(non B) and (non C) to be the most manageable one. So we
find ourselves invited to investigate the situations that are possible under

non B: each mathematician masters at most three languages, and

non C: each language is mastered by at most two mathematicians.

In the case non C, each mathematician communicates in different
languages with the other mathematicians he can communicate with; together
with non B, we conclude that each mathematician can communicate with
at most three other mathematicians. From this and the fact that the number
of mathematicians exceeds 4 we derive the existence of a pair of
mathematicians X and Y that cannot communicate with each other.
Mathematician X can communicate with at most 3 of the remaining 7, and
so can Y. Therefore, because 3 + 3 < 7, among those remaining seven there
exists at least one mathematician Z that can communicate neither with X,
nor with Y. The triple XYZ demonstrates the existence of a triple that is
incommunicado, hence the truth of A. (End of Solution.)

I have shown the above example for several reasons. It is a further
confirmation that the sooner the medieval relic "the counterpositive" is
forgotten, the better. (You see, in the case of "(A and B) => C"
we don't have a unique counterpositive! We have at least "(A and
non C) => (non B)", "(B and non C) =>
(non A)", and "(non C) => (non A or non B)". But is "(A) => (C or non B)" a
counterpositive as well?)

But a more disturbing observation can be made. Of course we could
conclude that the inclusion of this problem in the Mathematical
Olympics — traditionally solvable by an unusual solution requiring
very little specialized knowledge— was a mistake, because standard
heuristics lead to a straightforward solution. Indeed, the problem
would have been regarded as trivial, had it been phrased as folows:

"Nine mathematicians meet at an international conference. When it is
given that each mathematician masters at most three languages and
each language is mastered by at most two mathematicians, show that
there exists a triple of mathematicians, no two of which have a
language in common."

Composers of the Mathematical Olympics are, however, traditionally no
fools, and as a result I am disturbed by the observation that, evidently,
people are not supposed to see that the second phrasing states exactly the
same theorem as the first one. I can only draw two conclusions.

Firstly, it is not the theorem itself that presents any problems, it is only
the first formulation that should be regarded as a mystification. And, secondly,
the systematic demystification of such misleading formulations is in all probability
not a standard component of our mathematical curricula (for, if it had been,
the problem would never have been admitted). And that conclusion is a cruel exposure
of our standard mathematical curricula: if we don't teach how to avoid avoidable
complications our teaching must waste a lot of energy teaching clumsiness.

This is a very sad conclusion, but I am afraid that is is unavoidable.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

18th February 1980

prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by
eleftherios g stavridis
revised Wed, 20 Jun 2007
