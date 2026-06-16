---
title: "On non-determinacy being bounded. (EWD458)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD04xx/EWD458.html
published: "1974-10-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd04xx/EWD458.PDF
---

# On non-determinacy being bounded.

(A replacement of EWD416)

EWD458

On non-determinacy being bounded..

This is again a very formal chapter. In the chapter "Thecharacterization of semantics." we have mentioned four properties thatwp(S, R) , for any S considered as a function ofR , should have if its interpretation as theweakest pre-condition for establishing R is to be feasible.(For non-deterministic mechanisms the fourth one of these was a directconsequence of the second one.)In the next chapter "The semantic characterization of a programminglanguage." we have given ways for constructing new predicate transformers,pointing out that these constructions should only lead to predicate transformersenjoying the aforementioned properties (i.e. if the whole exercise is tocontinue to make sense). For every basic statement ("skip","abort" and the assignment statements) one has to verify thatthey enjoy the said properties; for every way of building up new statementsfrom component statements (semicolon, alternative and repetitive constructs)one has to show that the resulting composite statements enjoy those propertiesas well, in which demonstration one may assume that the component statementsenjoy them. We verified this up to and including the Law of the ExcludedMiracle for the semicolon, leaving the rest of the verifications as an exercise tothe reader. We leave it at that: in this chapter we shall prove a deeperproperty of our mechanisms, this time verifying it explicitly for the alternative and repetitive constructs as well. (And the structure of the latterverifications can be taken as an example,for the omitted ones.)Property 5. For any mechanism S and any infinite sequence of predicatesC0 , C1 , C2 ,... such that
for r ≥ 0:     Cr ⇒ Cr+1            for all states   (1)

we have for all states     wp(S, (E r: r ≥ 0: Cr)) =  (E s: s ≥ 0: wp(S, Cs))    .

(2)

For the statements "skip" and "abort" and for the assignment statements, the truth of (2) is a direct consequence of their definitions,assumption (1) not even being necessary. For the semicolon we derive wp("S1; S2", (E r: r ≥ 0: Cr)) =

(by definition of the semantics of the semicolon)wp(S1, wp(S2, (E r: r ≥ 0: Cr))) =

(because Property 5 is assumed to hold for S2 )wp(S1, (E r': r' ≥ 0: wp(S2, Cr'))) =

(because S2 is assumed to enjoy Property 2, so that wp(S2, Cr') ⇒ wp(S2, Cr'+1)and S1 is assumed to enjoy Property 5)(E s: s ≥ 0: wp(S1, wp(S2, Cs))) =

(by definition of the semantics of the semicolon)     (E s: s ≥ 0: wp("S1; S2", Cs))

QED.

For the alternative construct we prove (2) in two steps. The easystep is that the right-hand side of (2) implies its left-hand side. For,consider an arbitrary point X in state space,such that the right-hand side of (2) holds, i.e. there exists a non-negative value s', say, such that in point X the relation wp(S, Cs')holds. But becauseCs' ⇒ (E r: r ≥ 0: Cr) and anySenjoys Property 2 , we conclude that wp(S, (E r: r ≥ 0: Cr))

holds in point X as well. As Xwas an arbitrary state satisfying the right-hand side of (2),the latter implies the left-hand side of (2). For this argument, antecedent (1)has not been used, but we need it for proving the implication in the other direction.  wp(IF, (E r: r ≥ 0: Cr)) =

(by definition of the semantics of the alternative construct)BB and (A j: 1 ≤ j ≤ n: Bj ⇒ wp(SLj, (E r: r ≥ 0: Cr))) =

(because the individual SLj are assumed to enjoy Property 5)     BB and (A j: 1 ≤ j ≤ n: Bj ⇒ (E s: s ≥ 0: wp(SLj, Cs)))   .

(3)

Consider an arbitrary state X for which (3) is true, and let j' be a value for j such that Bj'(X) = true ; then we have in point X     (E s: s ≥ 0: wp(SLj', Cs))

(4)

Because of (1) and the fact that SLj' enjoys Property 2, we conclude that      wp(SLj', Cs) ⇒ wp(SLj', Cs+1)

and thus we conclude from (4) that in point X we also have     (E s': s' ≥ 0: (A s: s ≥ s': wp(SLj', Cs))   .

(5)

Let s'= s'(j') be the minimum value satisfying (5) . We now define smax as the maximum value ofs'(j') taken over the (at most n , and therefore the maximum exists!) valuesj' for which Bj'(X) = true. In point X then holds on account of (3) and (5) BB and (A j: 1 ≤ j ≤ n: Bj ⇒ wp(SLj, Csmax)) =

(by definition of the semantics of the alternative construct)wp(IF, Csmax)     .

But the truth of the latter relation in state X implies there also(E s: s ≥ 0: wp(IF, Cs))   ;

but as X was an arbitrary state satisfying (3) , for S = IF the factthat the left-hand side of (2) implies its right-hand side as well, hasbeen proved, and thus the alternative construct enjoys Property 5 as well.Note the essential role played by the antecadent (1) and the fact that aguarded command set is a finite set of guarded commands.

Property 5 is proved for the repetitive construct by mathematicalinduction.

Base:       Property 5 holds for H0.       H0(E r: r ≥ 0: Cr) =
(E r: r ≥ 0: Cr) and non BB =
(E s: s ≥ 0: Cs and non BB) =
(E s: s ≥ 0: H0(Cs))

QED.

Induction step:   From the assumption that Property 5 holds for Hk and H0 follows that it holds for Hk+1 .Hk+1(E r: r ≥ 0: Cr) =

(by virtue of the definition of Hk+1)wp(IF, Hk(E r: r ≥ 0: Cr)) orH0(E r: r ≥ 0: Cr) =

(because Property 5 is assumed to hold for Hkand for H0)wp(IF, (E r': r' ≥ 0: Hk(Cr'))) or(E s: s ≥ 0: H0(Cs)) =

(because Property 5 holds for the alternative construct and Property 2 isenjoyed by Hk)(E s: s ≥ 0: wp(IF, Hk(Cs))) or(E s: s ≥ 0: H0(Cs)) =
(E s: s ≥ 0: wp(IF, Hk(Cs)) orH0(Cs)) =

(by virtue of the definition of Hk+1)      (E s: s ≥ 0: Hk+1(Cs))   .

QED.

From base and induction step we conclude that Property 5 holds for all Hk , and hence wp(DO, (E r: r ≥ 0: Cr)) =

(by definition of the semantics of the repetitive construct)(E k: k ≥ 0: Hk(E r: r ≥ 0: Cr)) =

(because Property 5 holds for all Hk )(E k: k ≥ 0: (E s: s ≥ 0: Hk(Cs))) =

(because this expresses the existence of a (k, s)-pair)(E s: s ≥ 0: (E k: k ≥ 0: Hk(Cs))) =

(by definition of the semantics of the repetitive construct)      (E s: s ≥ 0: wp(DO, Cs))   .

QED.

*                *
*

Property 5 is of importance on account of the semantics of the repetitiveconstructwp(DO, R) = (E k: k ≥ 0: Hk(R)))   ;

such a pre-condition could be the post-condition for another statement. Because
for k ≥ 0: Hk(R) ⇒ Hk+1(R) for all states   ,

—this is easily proved by mathematical induction— the conditions under which Property 5 is relevant, are satisfied. We can, for instance, prove that in all initial states in which BB holds do B1 → SL1 ⌷ B2 → SL2 ⌷ ... ⌷Bn → SLn od

is equivalent toif B1 → SL1 ⌷ B2 → SL2 ⌷ ... ⌷Bn → SLn fi;
do B1 → SL1 ⌷ B2 → SL2 ⌷ ... ⌷Bn → SLn od    .

(In initial states in which BB, does not hold, the first program would haveacted as "skip", the secend one as "abort".) That is, we have to prove that (BB and wp(DO, R)) = (BB and wp(IF, wp(DO, R)))    .

BB and wp(IF, wp(DO, R)) =

(on account of the semantics of the repetitive construct)BB and wp(IF, (E k: k ≥ 0: Hk(R))) =

(because Property 5 holds for IF)BB and (E s: s ≥ 0: wp(IF, Hs(R))) =

(because (BB and H0(R)) = F )BB and (E s: s ≥ 0: wp(IF, Hs(R)) or H0(R)) =

(on account of the recurrence relation for the Hk(R) )BB and (E s: s ≥ 0: Hs+1(R)) =

(because (BB and H0(R)) = F )BB and (E k: k ≥ 0: Hk(R)) =

(on account of the semantics of the repetitive construct) BB and wp(DO, R)  .    QED.

*                *
*

Finally, we would like to draw attention to a very different consequence of the fact that all our mechanisms enjoy Property 5 . We could try to make the program S: "set x to any positive integer" with the properties:
a) wp(S, X > 0) = T

b) (A s: s ≥ 0: wp(S, 0 ≤ x < s) = F)    .

Here property a) expresses the requirement that activation of S is guaranteedto terminate with x equal to some positive value, property b) expresses thatS is a mechanism of unbounded non-determinacy, i.e. that no a priori upperbound for the final value of x can be given. For such a program S ,we could, however, derive now:T = wp(S, X > 0)
= wp(S, (E r: r ≥ 0: 0 ≤ x E s: s ≥ 0: wp(S, 0 ≤ x E s: s ≥ 0: F)
= F   .

This, however, is a contradiction: for the mechanism S: "set x to any positive integer" no program exists! As a result, any effort to write a program for "set x to any positive integer" must fail. For insiance, we could consider: go on:= true; x:= 1;
do go on → x:= x + 1
▯ go on → go on:= false
od    .

This construct will continue to increase x as long as the first alternativeis chosen; as soon as the second alternative has been chosen once, it terminatesimmediately. Upon termination x may indeed be "any positive integer" in thesense that we cannot think of a positive value X such that termination withx = X is impossible. But termination is not guaranteed either!We can enforce termination: with N some large, positive constant we can writego on:= true; x:= 1;
do go on and x od

but then property b) is no longer satisfied. The non-existence of a program for "set x to any positive integer"is reassuring in more than one sense. For, if such a program could exist, ourdefinition of the semantics of the repetitive construct would have been subjectto doubt, to say the least. With S: do x > 0 → x:= x − 1
▯ x od

our formalism for the repetitive construct gives wp(S, T) = (x ≥ 0) , whileI expect most of my readers to conclude that under the assumption of theexistsnce of "set x to any positive integer" forx termination would be guaranteed as well. But then the interpretation of wp(S, T) as the weakestpre-condition guaranteeing termination would no longer be justified. But whenwe substitute our first would-be implementation:S: do x > 0 → x:= x − 1
▯ x do go on → x:= x + 1
▯ go on → go on:= false
od
od

wp(S, T)= (x ≥ 0) is fully correct, both intuitively and formally. The second reason for reassurance is of a rather different nature: a mechanism of unbounded non-determinacy but yet guaranteed to terminate would be able to make within a finite time a choice out of infinitely many possibilities: if such a mechanism could be formulated in our programming language, that very fact would present an unsurmountable barrier to the possibility of the implementatlon of that programming language.

Acknowledgement. I would like to express my great indebtness to John C.Reynoldsfor drawing my attention to the central role of Property 5 and to the fact thatthe non-existence of a mechanism "set x to any positive integer" is essential for the intuitive justification of the semantics of the repetitive construct.He is, of course, in no way to be held responsible for any of the above. (Endof acknowledgement.)
Transcription by Moti Ben-Ari.
Revised Sat, 9 Dec 2006.
