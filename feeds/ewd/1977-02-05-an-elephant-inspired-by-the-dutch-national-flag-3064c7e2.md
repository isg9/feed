---
title: "An elephant inspired by the Dutch National Flag (EWD608)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD608.html
published: "1977-02-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD608.PDF
---

# An elephant inspired by the Dutch National Flag

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 608 An elephant inspired by the Dutch National Flag

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 264�267 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

An elephant inspired by the Dutch National Flag.
Encouraged by the success of EWD607, we now embark upon the analysis of a
more intricate elephant. We start with a cyclic arrangement of 3 + 3 mosquitoes.
Three main mosquitoes, called R(ed), W(hite), and B(lue) respectively, and three
buffer mosquitoes RW, WB, and BR, in between:
R → RW → W → WB → B → BR → R .
The buffer mosquitoes are quite simple, e.g.:

RW: begin channel W;

begin channel R; buf: pebble;

do R?(buf) → W!(buf) od

end

end

When its (input)channel with R ceases to exist, R?(buf) will become false,
and block exit will cause termination of the existence of the (output)channel with
W.
Each of the main mosquitoes has three “bags of pebble”, named “r(ed)”,
“w(hite)”,and “b(lue)” . The R mosquito must collect in its bag called “r”
all red pebbles in the system; its “foreign” pebbles it transmits, one at a time,
via the buffer mosquito RW , first emptying its blue bag because its blue pebbles
—that have to reach their destination via W — have to travel the longer distance.
The arrangement is worth investigating because we expect problems with the proof
of termination.
The solution that I am proposing has also a starting problem, but I am not
going to divulge that now: I hope that that difficulty emerges “naturally”
from a systematic analysis of our system.
mosquito R:

begin channel BR;

x, y: pebble; r, w, b: bag 3: pebble;

proc accept: if non BR?(y) → skip ▯ BR?(y) → place fi corp;

proc place: if white(y) → w:= w ≁ y ▯ red(y) → r:= r ≁ y fi corp;

r, w, := “initial values” {R3};

begin channel RW;

do card(b) > 0 → x:= any(b);’b:= b ≃ x;

RW!(x); accept

od {R2}:

do card(w) > 0 → x:= any(w); w:= w ≃ x;

RW!(x); accept

od

end {R1};                                    Note: “card” -short for “cardina1ity”—

do BR?(y) → place od {R0}             denotes “number of elements in”.

end                                                             (End of note.)

(and cyclicly).
We assume that —by some magic, not to be discussed here— BR (the text
of which starts with “begin channel R”) and R (the text of which starts with
“begin channel BR”) perform the entry to their outer blocks simultaneously,
thereby establishing the channel between them (which will be used only as an
input channel to R ). when the three input channels to the main mosquitoes have
been established, the six inner blocks will be entered —pairwise simultaneously,
but now R paired with RW — and the output channels for the main mosquitoes
have been established. (This is very informal and intuitive, but OK for the
moment: if coded wrongly, such paired block entries can, of course, create a
glorious deadlock.)
Let us now study mosquito R backwards. My final goal is to establish
proper termination with

R0:card(b) = card(w) = 0 and y-tai1(R0) is empty,

i.e. mosquito R has to terminate with red pebbles only when nothing will be
sent to it anymore; with “y-tail(Ri)” I denote the sequence of y-values still
to be absorbed in stage Ri before BR?(y) turns definitely false.
The first step is to investigate the transition from R1 to R0 .
Termination of the repetitive construct in between guarantees non BR?(y) , i.e.
guarantees that y-tail(R0) is empty; infinite repetition is excluded by
y-tail(R1) is finite .

Because card(b) = card(w) = 0 does not follow from “non BB” it had
better hold at R1 and be kept invariant by “place”. Keeping card(w) = 0
invariant by “place” implies the absence of white pebbles in the tail, avoiding
abortion implies the absence of blue ones, and we find for R1

R1:card(b) = card(w) = 0 and y-tail(R1) is finite and red only

Note The condition “finite and red only” is satisfied by the empty tail. (End
of’ note.)
The next step is to investigate the transition from R2 to R1 . Because
card(b) = 0 does not follow from “non BB” , we require it at R2 ; exclusion
of abortion taken into account:

card(b) = 0 and y-tail(R2) contains no blue pebbles

We have to impose more, because we have also to guarantee

card(w) = 0 and y-tail(R1) is finite and red only .

Termination guarantees card(w) = 0 and is guaranteed by

y-tail (R2) is finite

(For the variant function we can take: card(w) + number of white pebbles in y-tail.)
But how do we guarantee that y-tail(R1) is red only?
Let us define for a finite tail without blue pebbles

if tail contains no white pebbles:slack = - 1

if tail contains white pebbles 2 slack :the total number of red pebbles
preceding the last white one

and let us consider the relation card(w) > slack ; then

1)     card(w) = 0 implies that the finite tail is all red

2)     card(w) > slack is an invariant for the repeatable statement from
R2 to R1 ; because card(w) ≥ 0 by definition, this is obvious if the
resulting tail has no white pebbles, otherwise

2a) y has been white, in which case both’ card(w) and slack
remained unchanged

2b) y has been red, in which case both card(w) and slack have been
decreased by 1 .

Hence, collecting all our requirements , we deduce

R2:card(b) = 0 and y-tail(R2) is finite, without blue pebbles and
card(w) >-slack(R2) .

For the transition from R3 to R2 , infinite repetition is excluded a
priori, abortion is excluded by the absence of blue pebbles in the tail, the
invariant relation that does the trick is

card(b) + card(w) >-slack

and we find for R3

R3:y-tail(R3) is finite, without blue pebbles and eard(b) + card(w) > slack(R3) .

Taking the finiteness for a moment for granted, we see that

1)     the absence of blue pebbles in the y-tail is guaranteed (because R does
not transmit red pebbles, and cyclicly)

2)     slack(R3) ≥ 0 (because R does transmit blue pebbles, if any, before
white ones, if any, and cyclicly.)
Hence, a safe starting state is: each mosquito with at least one foreign
pebble! The complication at the start has, indeed, shown up nicely.
Termination was more easily demonstrated than originally feared.

1)     Mosquito R will generate in its x-sequence an a priori bounded number of
blue pebbles.

2)     In the same way, mosquito B will only generate in its x-sequence an
a priori bounded number of white pebbles.

3)     Equating the x-output of B with the y-input of R , we conclude that
mosquito R will only receive a bounded number of white pebbles.
Combining 1) and 3) we conclude that mosquito R will only generate a finite
x-sequence.
The proof of total conservation of pebbles is left to the reader.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

NL-4565 NUENENBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-12-13

.
