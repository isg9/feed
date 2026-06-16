---
title: "A nice theorem on monotonic predicate sequences (EWD818)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD818.html
published: "1982-04-03T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD818.PDF
---

# A nice theorem on monotonic predicate sequences

A predicate sequence Pi (i ≥
0) such that —denoting universal quantification over all states by
square brackets— (Ai :
i ≥ 0 : [Pi ⇒
Pi+1]) is called weakening; a predicate sequence
Pi (i ≥ 0) such that (Ai : i ≥ 0 :
[Pi+1 ⇒
Pi]) is called strengthening. A predicate sequence
Pi (i ≥ 0) that is
weakening or strengthening is called monotonic.

Theorem For any monotonic predicate
sequence Pi (i ≥ 0) we have

[(Ai : i
≥ 0 : (Ej : j
≥ i : Pj)) = (Ei : i ≥ 0 : (Aj : j ≥ i :
Pj))] .

Proof Let
Pi (i ≥ 0) be a weakening
predicate sequence; we then have for all natural i

(0)  [Pi = (Aj : j ≥ i :
Pj)]

(1)  [(Ej : j
≥ i : Pj) = (Ej : j ≥ 0 :
Pj)] .

For arbitrary Z we observe

[Z = (Ai :
i ≥ 0 : (Ej :
j ≥ i : Pj))]

= {(1)}

[Z = (Ai :
i ≥ 0 : (Ej :
j ≥ 0 : Pj))]

= {predicate calculus}

[Z = (Ej :
j ≥ 0 : Pj)]

= {renaming the dummy}

[Z = (Ei :
i ≥ 0 : Pi)]

= {(0)}

[Z = (Ei :
i ≥ 0 : (Aj :
j ≥ i : Pj))]

which proves the theorem for weakening Pi
(i ≥ 0).

Let Pi (i ≥ 0) be a
strengthening predicate sequence; then the sequence of predicates
¬Pi is weakening and we conclude
from the above

[(Ai : i
≥ 0 : (Ej : j
≥ i : ¬Pj)) =
(Ei : i ≥ 0 :
(Aj : j ≥
i : ¬Pj))] .

Negating both sides and applying de Morgan yields

[(Ei : i
≥ 0 : (Aj : j
≥ i : Pj)) = (Ai : i ≥ 0 : (Ej : j ≥ i
: Pj))] .

(End of Proof.)

As we see from the proof, the two equal expressions mentioned in the
Theorem are for a weakening predicate sequence
Pi (i ≥ 0) most simply
expressed as (Ei : i
≥ 0 : Pi); for a strengthening
predicate sequence Pi (i
≥ 0) they are most simply expressed as (Ai : i ≥ 0 :
Pi). These are the forms in which these
limits are best known.

*    *

*

C.S. Scholten and I discovered this theorem while working on EWD816. We
were slightly amazed that in its full generality the Theorem was new for
us, the more so because monotonic predicate sequences occur so frequently
and its proof is so simple. (The Theorem, though nice and in a way
striking, is definitely not deep.) Hence this note.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

3 April 1982

prof. dr. Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Matthew Hill

revised

11-Nov-2014
