---
title: "A terrible Thursday (EWD1151)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1151.html
published: "1993-01-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1151.PDF
---

# A terrible Thursday

EWD 1151

A terrible Thursday

Due to a conference on discrete algorithms, held here in town, the Department is flooded by speakers that "are in Austin anyhow". This morning, we had a speaker on the —I guess asymptotic— complexity of scheduling divide-and-conquer algorithms on a hypercube machine. He tried to convey his message via a mixture of sentences, gesticulations, formulae, and pictures; the design of a "language" adequate for the presentation of his material seemed to me to be his most urgent task. The end of his talk was particularly unintelligible; here I confine myself to something I saw earlier.

I had noticed that in almost all his formulae "p" occurred in the combination "log p", so that his formulae could be shortened by expressing them in y instead, with y and p linked by "y = log p". But then he showed the formula —well, not really!—

log p log logx p

Could this be expressed in y ? And, if so, how? The answer is "yes", and in the speaker's notation the result be

y logx y !

The explanation of this miracle is to be found in the intended parsing of the original formula, viz.

(log.p).(log.(log.p))x .

My question on how to parse his formula was considered irrelevant since, in the mean time, it had been shown that x=1 could be chosen.

*          *
*

Later that day we had a Distinguished Lecturer with a survey of the complexity results for approximations. There were obvious linguistic problems. Concerning the feasibility of something, he should have written

"There exists a δ for which it cannot be done"

or perhaps

"For some δ, it cannot be done"

or perhaps

"It cannot be done for all δ";

he wrote

"It cannot be done, for some δ",

in view of the preceding formulation clearly an unfortunate choice: it should be absolutely —i.e. syntactically— clear whether the clause about {delta} is within or without the scope of the negation, and the answer now depends on the absence or presence of a comma.

More suspicious I became at the visual stating that something

"maximizes the number of random strings R such that [...]"

where this was the only occurrence of the identifier R. This sentence was repaired by removal of the superfluous identifier "R" and the here nonsensical adjective "random". (Note that also the word "random" requires some sort of scope: when you introduce "a random bit" into something, this is short for introducing 2 somethings, one with a 0 and one with a 1. The scope of the duplication has to be clear.)

I became totally desperate at the introduction of something of the syntactic form

CPC [s(n),q(n),e(n)] .

Here, "n" was of type "natural number", because that was the type of the argument of functions s, q and e. I think that CPC was an identifier, global to the definition being given, that s, q and e were formal parameters —dummies, local to the definition— of type function, and that the mentioning of n was just a mistake —a convoluted way of typing s, q and e—. I think that an "instantiation" like

CPC [s(7),q(7),e(7)]

would be completely meaningless, because the something introduced should have been of the form

CPC [s,q,e] .

I underline "think" because I am not sure, since I could not get the clarification I sought. When the speaker gave the expression "log(n)" as example of a function, I gave up.

I had forgotten how incredibly and unforgivably sloppy the average mathematician is, how inconsistent about his syntax and how vague about the scopes of definitions and quantifications. I love mathematics, but it's mathematicians I cannot stand, for since ALGOL 60 there is no longer an excuse. By the time I came home, I was deeply depressed.

Austin, 28 January 1993

prof. dr. Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712–1188
USA

transcribed by Tristram Brelstaff
revised Wed, 7 Oct 2009
