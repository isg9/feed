---
title: "On Kleinrock’s Theorem (EWD775)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/ewd775.html
published: "1982-02-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD775.PDF
---

# On Kleinrock’s Theorem

The following has been triggered by C.A.R. Hoare’s “A general conservation law for queuing disciplines” (Information
Processing Letters 2 (1972) 82–85); this article refers to L. Kleinrock, “A conservation law for queuing disciplines”, Naval Research Logistics
Quarterly (June 1965) 181–192, and has been submitted because it offers for Kleinrock’s theorem a proof that is simpler than the original one.
In Hoare’s formulation, Kleinrock’s theorem is as follows. Assume (1) No server is idle when there is a waiting customer. (2) Arrival times and
service times of both customers and servers are not affected by the decision who is to receive service at what time
(i.e. they are independent of the scheduling). (3) Preemption is not used. Then

N
∑
i=1 Wi ·Ri

is independent of the choice of non-preemptive scheduling method. Here the customers are numbered from 1 through N, Wi
is the total waiting time for customer i, and Ri is the total service time for customer i.
This note is written to point out that (1) for a single-server system the proof is trivial, and (2) for a multi-server system the theorem
is not quite correct.
*              *
*

Consider a schedule in which customer q is served immediately after customer p, with Wq > Wp. The possible
decision to service these two customers in the other order would have affected the waiting time as follows.
Wp, Wq := Wp+ Rq, Wq− Rp
The products Wp· Rp and Wq· Rq are therefore increased and decreased respectively by the product of the service times, and
their sum remains constant.
Kleinrock’s theorem follows by induction

*               *

*

Consider a two-server system in which three customers, with service times =1, =1, and =2 respectively,
arrive simultaneously. Two schedules are possible with ∑Wi · Ri = 1 and = 2 respectively.
For customer i, which is served from t 0 till t1 we can define a “moment of inertia” of his service as

t1
∫
t 0 t ·dt
=
½
(t12−t 02)  =  (t1−t 0) · ⟮
t 0 + t1
/
2

⟯  =  Ri · (Ai + Wi  + ½ Ri)  =  Ri · Wi + Ci

where Ai is the moment of arrival of customer i and Ci = Ri · (Ai + ½Ri);
our assumptions imply that Ci is independent of the scheduling.
If S(t) is the number of customers served at moment t, then the "moment of inertia” of total service satisfies

T
∫
0 S(t) ·t ·dt
=
N
∑
i=1 Ri ·Wi
+
N
∑
i=1 Ci

Hence, Kleinrock’s theorem holds among schedules with the same “moment of inertia” of total service.
Acknowledgement. I am indebted to C.A.R. Hoare for having invited me to try to simplify is proof (which was already a
simplification of Kleinrock’s). (End of acknowledgement.)

Plantaanstraat 54 February 1982

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

12-Nov-2015

.
