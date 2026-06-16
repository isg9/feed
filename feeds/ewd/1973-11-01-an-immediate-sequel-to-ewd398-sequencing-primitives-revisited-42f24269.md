---
title: "An immediate sequel to EWD398: “Sequencing primitives revisited” (EWD399)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD399.html
published: "1973-11-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD399.PDF
---

# An immediate sequel to EWD398: “Sequencing primitives revisited”

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

An immediate sequel to EWD398: “Sequencing primitives revisited.”.
I have fixed the semantic definition of the if...fi and of the do...od
constructs as introduced in EWD398.
Let “Sif” be: if B1:S1 ▯ ... ▯ Bn:Sn fi ; then wp(Sif, P) =
{(E i: 1≤i≤n: Bi) and (1≤i≤n:(Bi and wp(Si, P)) or non Bi)} .
Let “Sdo” be: do B1:S1 ▯ ... ▯ Bn:Sn od ; then wp(Sdo, P) =
(E i: 0 ≤i: Hi(Sdo, P)) , where the Hi(Sdo, P) are given by the recurrence
relation:

H0(Sdo, P) = {P and non (E j: 1≤j≤n: Bi)}

for i >0:Hi(Sdo, P) = {wp(Sif, Hi-1(Sdo, P)) or Hi-1(Sdo, P)}

Here the “wp(Sif, ...) is the function defined above. The interpretation of
Hi(Sdo, P) is ”the weakest precondition such that we can guarantee
termination after at most i executions of a guarded command such that then the
postcondition P will be satisfied. It is indeed the weakest precondition,
if initially wp(Sdo, P) is not satisfied, either non-termination or
termination without establishing the truth of P or both are possible. In terms of
the Hi an alternative definition of the weakest precondition for the
do...od construct could have been —but I prefer to avoid the limit concept—

wp(Sdo, P) =lim Hi(Sdo, P)

i → inf

so we had better forget this again.
The decision to postulate —EWD398 - 4, last paragraph— “Fair random
selection” so that the construct as described on top of page 5 and in the
middle of page 8 is guaranteed to terminate, was a mistake: for such constructs
we prefer now not to exclude non-termination. It is just too tricky if the
termination —and in particular: the proof of the termination— has to rely
on the fair randomness of the selection and we had better restrict ourselves
to constructs were each guarded command, when executed, implies a further
approaching of the terminal state.

27th November 1973 prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

Plataanstraat 5

NUENEN - 4565, The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-11-08

.
