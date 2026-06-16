---
title: "On one-sided smoothing of event sequences (EWD478)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD04xx/EWD478.html
published: "1975-02-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd04xx/EWD478.PDF
---

# On one-sided smoothing of event sequences

NOTE: This transcription was contributed by Martin P.M. van der Burgt. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

On one-sided smoothing of event sequences.
We consider a finite number of events Ei (i from
0 through N) occurring at moments ti (i > j ⇒ ti > tj).
The problem I found myself faced with was how to
define a continuous function f(t) that could be
interpreted as “the event frequency at moment t.”
(I encountered this problem when trying to discover
a strategy for deciding how much primary store
should be allocated to independent programs in a
multiprogrammed environment when for each program
a “target page fault frequency” had been decided.)
A possible definition of the event frequency is
in terms of the Dirac function:

N

f(t) =
Σ δ (t - ti)(1)

i=0

which has the obvious advantage that it satisfies

+∞

∫ f’(t).dt = N + 1(2)

-∞

but the equally obvious disadvantage of severe
discontinuities, which makes the comparison of f0(t)
and f1(t) — two different frequencies corresponding
to different values of the parameter I could control —
rather difficult. Hence our desire to smooth the
data. The smoothing, however, had to be one-sided
in the sense that the smoothed value f’(t) may
at any moment t only depend on the set of
values ti with ti ≤ t. In order to make a
sensible choice among the wealth of possibilities
for f(t) we should impose upon f(t) as many
“sensible” constraints as we can think of. To
start with, property (2) seems a good choice.
The next one is that we should realize that
frequencies —as usually understood— have a
dimension “time -1”, i.e. if we consider at moments
t’i = ati and define the corresponding frequency
f’(t) for that second event sequence, then

f’(at) = (1/a) f(t)(3)

should hold. Poperty (3) is not in conflict with
requirement (2), for

+∞+∞+∞

∫ f’(t’).dt’ =∫ f’(at).d(at) =∫ a.f’(at).dt =

-∞-∞-∞

+∞

∫ f(t).dt = N + 1

-∞

Reasonable as requirement (3) may seem it is
(for finite sequences) too much to hope for: if t0 = 0
the relation (3) will not hold for t0 ≤ t ≤ t1, for how
are we going to guess “a”? Therefore we must loosen
it and can only require (3) to hold asymptotically
for t > ti with sufficiently large i.
Relation (2) however, still stands rather firmly. Because
in (2) we have only used

+∞

∫ δ(t) dt = 1

-∞

We could consider functions wi(t) such that

w(t) = 0 for t < 0(4)

∞

and
∫ w(t).dt = 1(5)

0

N

and replace (1) by f(t) =Σ wi (t - ti)

i=0

(6)

where (4) does credit to the one-sidedness of the
required smoothing , (5) guarantees (2) and (6) makes
the whole definition invariant for shifting of the
origin. The question is whether we can choose the wi
in such a way — satisfying (4) and (5) — as to
approximate (3) at least asymptotically.
A reasonable first guess — corresponding to fading
out of the past in an exponential way — for wi(t) is
with ki > 0:

wi(t) = 0 for t < 0

= ki.e-ki.t for t ≥ 0(7)

where we can try to adapt ki — on account of past
history, if any — towards a better approximation of (3).
I called (7) a first guess, because we also wanted
a continuous function f(t) and with (6) and (7),
f(t) is discontinuous for any t = ti. A continuous
f would follow from the proper linear compositum
— proper in view of (4), (5) and wi(t) = 0 —

wi(t) = 0 for t < 0

= (e-ki.t - e-hi.t).(hi.ki)/(hi - ki) for t ≥ 0(8)

and now we are free in the choice of both hi and ki,
provided that they are both positive and different
from each other. I am severely tempted to
restrict myself to

hi = c * ki with c ≠ 1 and c > 0(9)

the reason being that both hi and ki are of
dimension “time-1” and that I cannot forget
my target (3).
*              *
*

In order to come to grips with the constant c
I studied (omitting subscripts “i”) according to (8) & (9)

w(t) = k . c/(c-1) . (e-kt - e-ckt)

and tried to determine c so as to minimize the
maximum value of

c/(c-1) . (e-kt - e-ckt)

(I wanted my ripples as smooth as possible.) As this
led to the inacceptable value c = 1, I took the
limit for c → 1 and found — and this is my
final suggestion —

w(t) = 0 for t

= ki2t.e-kit for t ≥ 0(10)

∞

Also (10) has the property that∫ wi(t).dt = 1

0

*              *
*

The next thing to decide is, how to choose
the value ki. A constant value for ki seems
in view of (9) out of the question, because ki
has the dimension of a frequency. Apart from
initial difficulties it seems reasonable to choose
ki proportional to f(ti) i.e.

ki = d * f(ti)(11)

To get some feeling for a reasonable value
for d, we consider f(t) for t≥0 as results
from an infinite sequence of events that have
occurred at t=0, -1, -2, -3, .... etc; for all passed
events we may assume the same value ki = k
and we find for f(t), according to (6) and (10)

∞

f(t) = k.Σ k.(t+i).e-k.(t+i)

i=0

which, indeed satisfies f(0) = f(1) as was to
be expected.
Note. I summed the sequence and found

k2.e-k.t                 e-k

f(t) =(————) . (t + ————)

1 - e-k                  1 - e-k

(End of note)
The functions (10) have all one maximum,
viz. at t = ki-1. We would like to attribute
the maximum of f(t) for 0 ≤ t ≤ 1 to the term
with i = 0 (i.e. the last event). For i > 0 all the
terms should be decreasing functions of t for t > 0.
We can expect the smoothest maximum of f(t)
if it is in the neighbourhood of t = ½ and
this suggests k in the neighbourhood of 2 and

d = 2(12)

For, because in the average f(t) = 1, we know
that f(0) = f(1) < 1, therefore with d = 2 we
shall find k < 2 and the term with i = 0
will reach its maximum at t0 > ½. Because
the contribution of the remaining terms is a
decreasing function of t, this will be compensated
and f(t) will find its maximum at tf < t0.
Note. Using I sliderule I have investigated the
consequences of the choice d = 2. Salvo errore et
omissione I found for k=1.616, f(0) = f(1) = 0.808
as minimum value for f(t) and f(0.41) = 1.105 as
maximum value. Anyone that would like to rely on
these figures should reestablish them in a more
reliable manner, I only mention these results because
they represent the outcome of an hour’s work and give
some impression of how smooth f(t) becomes
in the case of equally spaced events. If it were
of importance, I would try slightly smaller values of
d as well. (End of note)

19th February 1975prof.dr.Edsger W. Dijkstra

Plataanstraat 5Burroughs Research Fellow

NUENEN - 4565

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-12-08

.
