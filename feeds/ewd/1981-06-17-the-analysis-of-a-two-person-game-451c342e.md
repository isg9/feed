---
title: "The analysis of a two-person game (EWD793)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD793.html
published: "1981-06-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD793.PDF
---

# The analysis of a two-person game

EWD793 - 0

The analysis of a two-person game.

As long as possible, two players make a move in turn; the player faced with a situation in which no move is possible has lost the game.

The situation is given by two natural numbers (i.e. integers ≥0). A move being possible means that both numbers are positive; a move then consists in decreasing one of the numbers one or more times by the other, but such that the number decreased remains ≥0.

It as asked to characterize those initial situations in which the player who has the first move can guarantee to win.

* * *

Our first observation is that, since no move affects the greatest common divisor of the two numbers, only their ratio matters. Our second observations is that, since the situation is symmetric in the two numbers, we can confine ourselves when moves are possible to a ratio ≥1. Our third observation is that, to start with, a move consists in decreasing the ratio by an integer; if then 0 < ratio < 1, the roles of smallest and largest number have been interchanged, and we should replace the ratio by its inverse. And this suggests to characterize the ratio by its finite expansion as continued fraction, more precisely by the sequence of positive integers

EWD793 - 1

(0)k0, k1, ... , kn

such that the ratio equals rn, given by

r0 = k0 and ri+1 = ki+1 + 1 / ri    .

A move with a non-maximal decrease decreases kn by an amount < kn ; a move with a maximal decrease decreases n by 1 ("drops kn").

The state n=0 ∧ k0=1 corresponds to a ratio =1, which is a winning situation for the initial player. It is exceptional in the sense that all ratios >1 can be represented with k0 ≥2 . For such states we define

trail = n - ( MAX i : 0≤i≤n ∧ ki ≥2 : i )

i.e. the length of the train of ones on which (0) ends. Handing your opponent a situation with trail odd forces him to return a situation with trail even. With trail even you can either hand your opponent a situation with trail odd or terminate the game and win. Hence trail even characterizes the remaining winning situations, i.e. the ratio > (1 + √5) / 2 .

Plataanstraat 5

5671 AL NUENEN

The Netherlands

17 June 1981

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on Sun, 13 Jul 2003.
