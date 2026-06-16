---
title: "A small note on the additive composition of variant functions (EWD592)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD592.html
published: "1976-11-07T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD592.PDF
---

# A small note on the additive composition of variant functions

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

A small note on the additive composition of variant functions.
This is only a small note, for its observation is nearly trivial. The
note is made, however, because it describes a method of some generality, by
which results can be derived that —as I have observed— tend to strike the
reader as “a trick” if he has never seen the like and is unaware of the method
behind it.
It deals with finding a variant function for a repetitive construct with
more than one alternative:

do B1 → S1 ▯ ... ▯ Bn → Sn od .

The simple idea is to contruct the variant function t as the sum

t = t1 + ... + tn ,

where ti is decreased by at least one by the i-th guarded command and left
constant by the others.
We shall illustrate it with an example of matching simplicity. On a
shunting yard are a finite number of trains; a train is either atomic (iff it
consists of a single car) or composite (if? it consists of a finite number of
cars greater than one). The yard is empty, when it contains no trains.
Repeatedly we select a train: if it is atomic, it is removed from the yard, if it is
composite, it is split into two (shorter) trains that, in this move, remain on
the yard. The process terminates, when the yard is empty. The point is to
construct the decreasing function t that is bounded from below, so that
termination is proved. We sketch the process by

do atomic train selected → remove it

▯ composite train selected → split it

od

With CARS = the number of cars on the yard

t1 = CARS

satisfies the requirement that the first guarded command decreases it and the
other one leaves it constant. with

TRAINS = the number of trains on the yard ,

t2 = - TRAINS

would satisfy the requirement that it is decreased by the second guarded commend
(because that increases + TRAINS). This remains true when we add a function
left constant by it, e.g.

t2 = f(CARS) - TRAINS .

(Note that we have already verified that CARS , and therefore f(CARS) for any
function f , is unaffected by the second guarded command.) Because t2
has to remain constant when the first guarded command is selected, we must
choose for f the identity function:

t2 = CARS - TRAINS

and thus we find t = 2 * CARS - TRAINS .

(With CLOSCONNS = the number of closed car-to-car connections,

t2 = CLOSCONNS

also does the job: it is left unchanged by the first and decreased by the second
guarded command. This is not a new solution, for CLOSCONNS = CARS - TRAINS . )
*              *
*

For the asymmetric computation for x = gcd(X, Y) with X > 0 and Y > 0 :

x, y := X, Y; do X > y → x:=x - y ▯ y > x → x, y == y, x od

we find quite analogously t1 = x + y and t2 = y .
*              *
*

Note. Obviously the condition that ti is left unchanged by the j-th (j ≠ i)
guarded commands is stronger than necessary: it suffices to establish that ti
is not increased by the other guarded commands. (End of note.)
Note. Instead of choosing t = t1 + ... + tn , we could also take —after
having chosen the ti — t = a1*t1 + ... + an*tn with all ai’s positive;
in other words, the variant function is by no means unique, and we can choose
the most convenient one. (End of note.)
The above has been prompted by my impression that some people seem to think
that that there is something mysterious about the termination of the type of
programs discussed. Its moral is that there is no mystery.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

NUENEN NL-4565Burroughs Research Follow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-12-08

.
