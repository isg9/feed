---
title: "A computing scientist’s approach to a once-deep theorem of Sylvester’s (EWD1016)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1016.html
published: "1988-02-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1016.PDF
---

# A computing scientist’s approach to a once-deep theorem of Sylvester’s

EWD 1016

A computing scientist's approach to a once-deep theorem of Sylvester's

Well, actually it wasn't Sylvester's theorem, it was only his
conjecture —dating from the year 1893—, and it remained so for more
than 40 years until T. Gallai (alias Grunwald) "finally succeeded,
using a rather complicated argument" [Coxeter]. We shall derive
(essentially) the simple argument of L.M. Kelly (1948).

Theorem  Consider a finite number of distinct points in the real Euclidean plane; these points are collinear or there exists a straight line through exactly 2 of them.

To see that this is truly a geometric theorem and not a combinatorial one, we slightly rephrase the setting. Let us assume for a moment that collinearity of points in the Euclidean plane is fully captured by

(i) any pair of points are collinear, and if two collinear triples have two points in common, their four points are collinear.

Let us rephrase this: replace "points" by "people", "lines" by "clubs" and "collinear" by "club-sharing" (i.e., belonging to the one and the same club). Club membership is postulated to satisfy the analogue of (i):

(ii) any pair of people is club-sharing, and if two club-sharing triples have two people in common, their four people are club sharing.

The analogue of Sylvester's theorem would state for a finite
population: all people belong to one and the same club or there exists
a club with exactly 2 members. Its falsity is shown by the following
counterexample of 7 people —numbered from 0 through 6— with the
following 7 clubs: {013}, {124}, {235}, {346}, {450}, {561}, and
{602}. Postulate (ii) is satisfied because each of the 21 pairs of
people occurs in exactly 1 club. Thus we have established that
Sylvester's theorem is truly a geometrical one; let us now try to
prove it.

Being computing scientists, we love constructive arguments, i.e. we
like to show that something exists by designing an algorithm that
computes such a thing. We therefore propose to design an algorithm
that computes a line that passes through exactly 2 of the points from
a given finite, non-collinear set of distinct points. (Legenda: from
here on we no longer repeat that the points are distinct, nor that
they belong to the given, non-collinear set.)

More precisely, we have to design an algorithm that operates on a variable q of type: line and that establishes the post-condition R, given by

R : q passes through exactly 2 of the points.

The simplest idea is to initialize q by the line through 2 arbitrary points. (This is always possible because, the given set being non-collinear, there are at least 3 points.) If q goes through 3 or more points, it has to be changed, otherwise it can be accepted as final value. That is, with invariant P given by

P : q passes through ≥ 2 points

we propose as first approximation of our algorithm

establish P by initialization of q
; do q passes through ≥ 3 points →
change q under invariance of P
od

Because

P ∧ ¬ (q passes through ≥ 3 points)   ⇒   R

we are done when the algorithm terminates.

Our remaining task is to ensure that it does terminate. To that end we have to exploit the finiteness of the given set and its non-collinearity. Because the exploitation of finiteness is absolutely standard, we first focus our attention on what we can conclude from the non-collinearity. From the latter we can draw only one conclusion in connection with q, viz. the existence of a point through which q does not pass. That is, we propose to introduce a variable E of type: point, and to strengthen P to P1

P1: q passes through ≥ 2 points ∧
q does not pass through E.

The new approximation of our algorithm is

establish P1 by initialization of q and E
; do q passes through ≥ 3 points →
{?} change q under invariance of P1
od

(Ignore for a moment the assertion "{?}"; the important thing to realize is that with the feasibility of maintaining the stronger P1, the non-collinearity of the given set has been exhausted.)

In the current stage of program design, our only option is a further refinement of the as yet rather nondeterministic
(0)     "change q and E under the invariance of P1"

Because we may have to reduce its nondeterminism lest the algorithm fails to terminate, let us investigate its freedom: what precondition {?} can we guarantee? We know of the existence of 4 points, viz. E and the three points on the current q. Because the new q has to pass through ≥ 2 points and has to differ from the old q, the new q passes through the old E and one of the 3 points on the old q; in each case, one of the remaining 2 points on the old q has to be chosen as the new E. In summary: for the new pair (q, E ) we have 6 possibilities.

For the termination argument we need a variant function of the pair (q, E ); because the number of points is finite, the number of pairs (q, E ) satisfying P1 is finite, and any function of the pair (q, E ) that decreases at each change will do.

What is the simplest function of a line and a point (not on that
line) that we can think of? The Euclidean distance between the
two!

Let us investigate whether we can refine (0) so as to decrease the distance between q and E. Let us name the three points on the old q: A, B, C, so that A becomes the new E. With that convention, the refinement of (0) that decreases the distance of the pair (q, E ) as much as possible is

(1)    q, E := of BE and CE the nearest to A, A.

Finally we derive a condition on A as our choice for the new E from the requirement that the variant function decreases. With

h = distance between E and q

b = distance between A and BE

c = distance between A and CE

the required decrease of the variant function is expressed by

(2)    b min c < h

In order to derive (2), we proceed as follows

b min c  <  h

=        { definition of min }

b<h  ∨  c<h

=        { similar triangles, see figure }

AB<EB  ∨  AC<EC

⇐        { monotonicity of + }     (See Note)

AB+AC  <  EB+EC

⇐        { P1 ⇒   BC < EB+EC, i.e. the strict triangle inequality }    (See Note)

AB+AC ≤ BC

=        { AB+AC ≤ BC, i.e. trianglular inequality }

AB+AC = BC

=        { AB, AC and BC denote unsigned lengths }

on q, A lies between B and C.

Hence, with for A the point between the two others, (1) does the job. And this concludes our proof of Sylvester's theorem.

Note Since steps that express equivalence don't
destroy information, the others need some more justification. We all
know the monotonicity of ≥ , i.e. no one doubts

x≥x' ∧ y≥y'   ⇒  x+y ≥ x'+y';

its counterpositive yields the equivalent

x<x' ∨ y<y'   ⇐   x+y < x'+y'

and that is what we used.

To justify the next step, a second look at our demonstrandum (2) suffices: since it is impossible to demonstrate (2) for h=0, we have to use once more that q does not pass through E. Since E occurs in this calculation in the combinations EB and EC, we translate this into the non-degeneracy of triangle BEC. The step eliminates E for the rest of the calculation. (End of Note.)

For the sake of completeness we point out that, after the choice of
the variant function, we have made two silent choices. We have chosen
—as usual— to decrease the variant function; because of the finiteness, successfully increasing it would also have yielded a valid termination argument. With a huge h and ABC close together, however, the distance between E and q cannot be increased, which settles this silent choice. Moreover, we could have grouped our 6 cases differently, viz. by common new q instead of by common new E. We could have said "Let us name the three points so that AE becomes the new q" and instead of (1) we would have come up with

q,E := AE, of B and C the nearest to AE.

It does not work.

*          *

*

A few methodological remarks are in order, because the theorem certainly deserves the name of a once-deep theorem: after 1933, when Karamata and Erdös revived interest in the problem [Coxeter], it took another fifteen years before L. M. Kelly made essentially the above use of the Euclidean distance and found the simple argument.

It was very gratifying to see that, once the decision had been taken to tackle this problem as a programming task, the job of designing the program was all but standard. I have used this problem in oral examinations for a course on "Mathematical Methodology" for Computing Science graduates. Some needed more prompting or more time than others, but none of them needed any prompting to come up with the Euclidean distance between q and E as candidate for the variant function. They knew that the argument required a variant function and they all suggested the Euclidean distance without hesitation. And that was Kelly's great invention!

A fringe benefit of proving the theorem by designing a program is
that it takes away the surprise that in such a non-metric context a
metric concept such as the Euclidean distance enters the picture. We
all know that a monotonic function of an acceptable variant is again
an acceptable variant and that the challenge always is to find a nice
one. It is very much like the freedom to chose the most convenient
coordinate system.

Acknowledgement That the distance from points to lines through other points could be used in the proof was told to me by Bernhard von Stengel; he told me to look at the shortest such distance. (End of Acknowledgement.)

*          *

*

For Sylvester's theorem to be true it is essential that the points
are distinct. (Consider a non-degenerate triangle with each vertex
coinciding with a triple of points.) Replace "any 2 points are
distinct" by "through any 2 points passes only one straight line".
The latter can be generalized to one dimension more "through any 3
points passes only one plane", i.e. "no 3 points are collinear". And
now we are ready to generalize Sylvester's theorem to three
dimensions.

Theorem   Consider in the real three-dimensional Euclidean space a finite number of points such that no 3 of them are collinear; these points are coplanar or there exists a plane through exactly 3 of them.

(In the three-dimensional case, just requiring that the points be
distinct is not enough: consider two non-intersecting, nonparallel
lines with 4 distinct points on each.)

Proof sketch  Select one of the points and a plane outside. Project, with the selected point as center, the remaining points on the selected plane, to which projection Sylvester's theorem is applied. (End of proof sketch.)

*          *

*

Coxeter mentions no more-dimensional generalization of Sylvester's
theorem. Maybe he did not think it sufficiently interesting. Maybe
he could not comfortably generalize because none of his formulations
mentions explicitly that the points have to be distinct.

Coxeter's passage is interesting for historical reasons. He quotes
Sylvester's original statement of the problem:

Prove that it is not possible to arrange any finite number of real points so that a right line through every two of them shall pass [sic] through a third, unless they all lie in the same right line.

Fortunately we don't formulate problems like that anymore. I could not read it and ended up looking up "unless" in the COD, which gave two interpretations "if not" (which boils down to ∨) and "except when" (which boils down to ≢). I felt excused. (Actually, Sylvester used "unless" in the meaning ∨).

A few lines further, Coxeter gives credit to T. Motzkin (1951):

Sylvester's 'negative' statement was rephrased 'positively' by Motzkin:

If n points in the real plane are not on one straight line, then there exists a straight line containing exactly two of the points.

It is not quite clear for which achievement Motzkin receives credit.
He replaces Sylvester's contorted

¬(∀ line: line passes ≥ 2: line passes ≥ 3 )

thanks to de Morgan by

(∃ line: line passes ≥ 2: line passes < 3 )

which can be simplified (arithmetically!) to

(∃ line:: line passes = 2 )

which in the statement of the theorem is certainly a simplification.
(But note that in the proof one immediately uses

line passes =2   ≡   (line passes ≥ 2) ∧ ¬(line passes ≥ 3). )

Or did Motzkin get credit for replacing Sylvester's disjunction by an implication? You never know with Coxeter. (I dislike in Motzkin's formulation, besides the dangling n, the implication: compared to Sylvester's disjunction, I consider that a step backwards. Please note that the implicative formulation introduced —in "not on a straight line"— a negation.)

Coxeter's section opens with a quotation from G.H. Hardy (1940):

Reductio ad absurdum, which Euclid loved so much, is one of a mathematician's finest weapons. It is a far finer gambit than any chess gambit: a chess player may offer the sacrifice of a pawn or even a piece, but a mathematician offers the game.

No matter how hard I try, almost half a century later I am unable
to give even a mild sensible interpretation to the above quotation of
Hardy's, but, in 1961, Coxeter evidently felt he could: his proof
gloriously ends with the now infamous "which is absurd".

(Coxeter focuses his attention on the pair (q, E ) with minimum distance and derives a contradiction from the assumption that q passes through at least 3 points. The choice of the pair with minimum distance is overspecific: it is only a device to construct the avoidable contradiction. Tellingly, he concludes

"This completes the proof that there is always a line containing exactly two of the points. Of course, there may be more than one such line [...]".)

Fascinating to analyze mathematical style from such a recent past!

Reference

Coxeter, FRS, H.S.M., Introduction to Geometry, 2nd Ed., John Wiley & Sons, Inc., New York etc., pp 65–66.

Austin, 5 February 1988

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Xiaozhou (Steve) Li

revised Sun, 9 Dec 2007
