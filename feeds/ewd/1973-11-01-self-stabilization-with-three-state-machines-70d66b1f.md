---
title: "Self-stabilization with three-state machines (EWD396)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD396.html
published: "1973-11-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD396.PDF
---

# Self-stabilization with three-state machines

I consider N+1 machines numbered from 0 through N (N > 1). In order to avoid avoidable subscripts in my text, I introduce for each machine nr.i a local terminology

L = state of machine nr.(i–1)mod(N+1), its “lefthand neighbour”

S = state of machine nr.i (S from “self”)

R = state of machine nr.(i+1)mod(N+1), its “righthand neighbour”.

For each machine one or more privileges are defined, i.e. boolean functions of L, S and R. A “move” consists of selecting one of the privileges currently present and bringing the machine enjoying it in a new state, i.e. S:= f (L,S,R), where the function f may depend on the privilege selected in the case of a machine for which more than one privilege has been defined.

We call a state of the total system “legitimate” if and only if

1)     exactly one privilege is present and, therefore, exactly one move is possible, and

2)     the execution of the only possible move will again bring the total system in a legitimate state, and

3)     successive moves will cause each privilege to be present with equal frequency.

We are interested in designing a set of finite state machines, privileges and moves, such that, in addition, the system enjoys the property of “self-stabilization”, i.e. regardless of the initial state and regardless of the privilege selected for the next move, it must always be possible to do a next move and after a finite number of moves the total system must arrive at a legitimate state.

Each machine is a three-state machine, its state being denoted by 0, 1 and 2; in the following “+” and “–”, when referring to machine states, are to be understood as operations, reduced mod 3 .

The machines nr.i with 0 i N are called “the normal machines”; they have two privileges:

the privilege   “S + 1 = L” allows the move  “S:= S + 1”  and

the privilege   “S + 1 = R ” allows the move  “S:= S + 1”.

Machine nr.0 —the so-called bottom machine— has one privilege:

the privilege  “S + 1 = R ”   allows the move  “S:= S – 1”.

Machine nr.N —the so-called top machine— has also only one privilege:
the privilege  “L = R and S ≠ L + 1”  allows the move  “S:= L + 1”.

We observe

0)     each move destroys the privilege allowing it

1)     at least one privilege must be present (no privilege among the bottom machine and the normal machines implies that they are all in the same state; if the top machine is not to have its privilege, it must be on account of  S = L + 1, which contradicts the assumption that its normal neighbour had no privilege)
2)     between two successive moves of the top machine, the bottom machine must have moved at least once.

We now consider the N differences between nr.i and nr.i+1 (0 ≤ i N) and write a “+” between the two of each pair if the lefthand state is one higher than the righthand state, a “–” if the lefthand state is one less than the righthand state and nothing if they are equal. If the normal machine B does a move, we have five possibilities
a)     “A + B   C ” becomes “A   B + C ” (a “+” travelling to the right),

b)     “A   B – C ” becomes “A – B   C ” (a “–” travelling to the left),

c)     “A + B + C ” becomes “A   B – C ” (internal reflection of “+ +”)

d)     “A – B – C ” becomes “A + B   C ” (internal reflection of “– –”)

e)     “A + B – C ” becomes “A   B   C ” (mutual annihilation of “+ –”).

Transitions c), d) and e) reduce the total number of “+” and “–” signs.

A move of the bottom machine requires a “–” to the right of it and changes it in a “+”: it is a true reflection, leaving the number of “+” and “–” signs unchanged. A move of the top machine creates a “–” at the left of it —in doing so it may absorb a “+” in that position—, but on account of observation 2) it cannot do so in a higher frequency than the bottom machine absorbs “–” signs.

In the legitimate states there is only one “+” or “–” sign, travelling in the appropriate direction and being changed —and then reflected— at the ends.

Self-stabilization is now obvious. A move of the bottom machine absorbs a “–” gone to the left and produces in its place a “+” ready to go to the right. How is the bottom machine going to receive its next “–” in the non-legitimate state? Either because the right going “+” meets a pair “– –” (“+ – –” can reduce to “–”) or because it meets another “+” (“+ +” can reduce to “–”) that could have been created by “– –”. In order to stay away permanently from the legitimate state, the top machine would have to create two “–” signs for every single “–” arriving at the bottom machine. But this possibility is denied by our observation that between two successive moves of the top machine, the bottom machine must have moved at least once.

Note. In view of the fact that a three-state machine requires two bits, with which we can have four-state machines, for whom the problem of self-stabilization has already been solved, the result achieved here could very well be totally insignificant. This aspect of minimization of the number of machine states has reached the stage of a problem in pure mathematics and I shall not pursue it any further.

1st November 1973                                              prof.dr.Edsger W.Dijkstra

BURROUGHS                                                     Research Fellow

Plataanstraat 5

NUENEN - 4565

The Netherlands

transcribed by Chrys Alexander Panayiotou

revised 08-Sep-2013
