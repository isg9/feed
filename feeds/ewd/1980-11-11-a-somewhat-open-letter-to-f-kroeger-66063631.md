---
title: "A somewhat open letter to F.Kroeger (EWD760)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/ewd760.html
published: "1980-11-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD760.PDF
---

# A somewhat open letter to F.Kroeger

A somewhat open letter to F. Kr�ger.
Dear Dr. Kr�ger:
In a recent article [1], you prove —in my notation, see [2] —

{a ≤ 100 ∧ b = 1}

do 100 < a ∧ b ≠ 1 → a, b := a−10, b−1

⫿ a ≤ 100 → a, b := a+11, b+1

od {a = 101}

Allow me to show you the argument I, as a programmer, came up with.

I rewrote the proof obligation as follows

{a ≤ 100 ∧ b = 1}

do a ≤ 100 → a, b := a+11, b+1 od;

{P0, hence P0 ∨ P1 ∨ P2}

do 110 < a ∧ b ≠ 1 → a, b := a−10, b−1 {P0 ∨ P2}

⫿ 100 < a ≤ 110 ∧ b ≠ 1 → a, b := a−10, b−1 {P1}

⫿ a ≤ 100 → a, b := a+11, b+1 {P0}

od {(P0 ∨ P1 ∨ P2) ∧ 100 a ∧ b =1, hence a = 101}

withP0 = 100 < a ≤ 111 ∧ b ≥ 2

P1 = 90 a ≤ 100 ∧ b ≥ 1

P2 = (a = 101) ∧ (b = 1) .

The two program transformations have no effect. You may always insert before a repetitive construct an excerpt from it; you may
always split a guarded command. The first repetitive construct terminates obviously. The verification that P0 ∨ P1 ∨ P2
is an invariant of the second one is simple and straightforward.
In order to prove the termination of the second repetitive construct, we observe

1)
the initial truth of P0. (excluding truth of the last guard)

2)that the truth of P1 implies the falsity of the first two guards and the truth of the last one.

As a result
the last two guarded commands may be merged and the invariant relation may be strengthened. We rewrite the second repetitive construct as follows

{P0, hence P0 ∨ P2}

do 110 < a ∧ b ≠ 1 → a, b := a−10, b−1 {P0 ∨ P2}

⫿ 100 < a ≤ 110 ∧ b ≠ 1 → a := a+1 {P0}

od {(P0 ∨ P2) ∧ b = 1, hence a = 101}

In view of P0 ∨ P2, the first guarded command sets a = 101; the second guard remains true until a = 111. The guards
being mutually exclusive, we may rewrite —note that a loop with only the second guarded command obviously terminates—

{ P0 ∨ P2 }

do 110 < a ∧ b ≠ 1 → a, b := 101, b−1 {P0 ∨ P2}

⫿ 100 < a ≤ 110 ∧ b ≠ 1 → a := 111 { P0 }

od

Because P0 ∧ a = 111 implies the truth of the first guard, by almost the same device as used before,
the two guarded commands can be merged, giving

{ P0 ∨ P2 }

do 100 < a ∧ b ≠ 1 → a, b := 101, b−1 {P0 ∨ P2} od

which trivially terminates. This includes a proof with a minimum amount of formal labor.

*              *
*

When I saw the length of your proof, it put me off; I stopped reading and developed the above argument. I had never really
analyzed with McCarthy’s 91 function: it has always struck me as a rather contorted puzzle. After the above proof, I proved the
termination in the style of R.W. Floyd, using a well-founded set; from that I derived a closed formula for the exact number of
repetitions. It is neither long, nor difficult; but it is too boring to record here; the steps of the derivation are completely standard.
*              *
*

In my opinion —but the opinion is a well-considered one— computing science needs the kind of vigorous mathematics that results
from a subtle balance between the application of formal techniques and of common sense. (And there are reasons to believe that
in computing science that balance is more critical than in many other areas of mathematics.)
I trust you can now understand why I find it hard to consider your “Generalized invariants” of great significance in the context
of computing science. Perhaps your paper had better been published in the Journal of Logic than in Acta Informatica, where
it gives fuel to the accusation that the latter publishes too much “pompous irrelevance”.

[1]Kr�ger, F. : Infinite proof rules for loops. Acta InInformatica 14, 371-389 (1980)

[2]
Dijkstra, Edsger W.: A Discipline of Programming. Prentice-Hall, Inc, Englewood Cliffs, N.J., USA

Plantaanstraat 511 November 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
