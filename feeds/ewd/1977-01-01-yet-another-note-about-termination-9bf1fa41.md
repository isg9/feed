---
title: "Yet another note about termination (EWD597)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD597.html
published: "1977-01-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD597.PDF
---

# Yet another note about termination

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Yet another note about termination.
I had never thought that I would have to write this note, but apparently
I have to. In a paper I had written:

“For each repetitive process we must have a monotonicity argument on
which to base our proof of termination.”

To my utter amazement, the Editor of the journal to which it was submitted.
expressed in his letter seven lines of severe doubt about the above statement,
ending with: “Perhaps it is true, but it is a rather sweeping claim.” I could
only conclude that the need for a monotonicity argument for termination proofs
is not the common knowledge I supposed it to be, and that the need, even when
stated, is not intuitively obvious to everybody.
Because perhaps my Editor’s hesitation was caused by the nondeterminacy
that also played a role in that paper, let us consider the general —and in
general nondeterministic— repetitive construct DO:

do B1 → S1 ▯ ... ▯ Bn → Sn od .

On my recent trip through the USSR, Tony Hoare, who is presently more
operationally inclined than I, described it as

{if B1 →S1 ▯ ... ▯ Bn → Sn fi}* or {IF}*

where with {...}* he meant “as many successive executions as possible”,
where the execution of IF is regarded when all the guards are false (non BB).
A termination proof for a given state x means that after a finite number of
steps —i.e. applications of IF — the state non BB is reached. The actual
number of steps may not be determined by x , e.g. do y ≥ 2 → y=: y - 2 ▯
y ≥ 1 → y:= y - 1 od with initially y ≥ 2 ; guaranteed termination for the
initial state x , however, means that the maximum number of steps needed
is finite. Denoting this maximum number by mn(x) , termination is guaranteed
in all points x where mn(x) is finite. Denoting the transformation of
the state as effectuated by IF symbolically by x=: F(x) —where F may
be a partial function: F(x) need not be defined for states x , in which
BB does not hold— it is clear that with the above meaning of mn we must
have

(mn(x) > 0) ⇒ (mn(F(x)) ≤ mn(x) -1) .

Note. If F(x) is a single valued function, the program is deterministic and
the maximum number of steps needed is also the actual number of steps needed;

and then we have

(mn(x) > 0) ⇒ (mn(F(x)) = mn(x) - 1) . (End of note.)

Hence, for all initial states in which termination is guaranteed to occur,
the finite function mn(x) which decreases monotonically by at least one at
each application of x:= F(x) exists; conversely: each proof of termination
boils down to a proof of the existence of such a monotonically decreasing
function.
*              *
*

Because my challenged claim is one about all possible arguments for
termination, a less operational approach that is directly based on the axiomatic
definition of the repetitive construct might be appreciated.
For the repetitive construct DO the predicate transformation —see
[1], page 35— is given in terms of the predicate transformation of the
corresponding alternative construct IF by

H0(R) = R and non BB

Hk+1(R) = wp(IF, Hk(R)) or H0(R)(2)

wp(DO, R) = (E k: k ≥ 0: Hk(R)) .

From (2) it follows that for i < j we have Hi(R) ⇒ Hj(R) for all states
and all post-conditions R . Proving termination means showing that a point
x satisfies wp(DO, T) , i.e. that there exists a value t(x) such that
Hk(T) holds there for all values of k satisfying k ≥ t(x) . As x
satisfies an invariant relation, P say, it suffices to prove that

(P and t ≤ k) ⇒ Hk(T) for all states x

But this is the same formula as on the top of [1], page 42. In view of the
fact that (2) defines Hk+1 in terms of Hk , mathematical induction over k
is by definition the only available pattern of reasoning. The argument is
carried out in [1], pages 41/42; it shows quite clearly how for the
transition from k to k+1 the monotonicity assumption about IF , viz.

(P and BB and t ≤ t0+1) ⇒ wp(IF, t ≤ t0)

is essentially needed.
[1] Dijkstra, Edsger W., “A Discipline of Programming”. Prentice Hall, 1976

Plataanstraat 5prof.dr.Edsger W.Dijkstra

NL-4565 NUENENBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-12-12

.
