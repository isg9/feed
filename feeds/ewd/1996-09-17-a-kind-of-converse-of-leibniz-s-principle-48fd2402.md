---
title: "A kind of converse of Leibniz’s Principle (EWD1245)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1245.html
published: "1996-09-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1245.PDF
---

# A kind of converse of Leibniz’s Principle

by

Edsger W. Dijkstra

We owe to Gottfried W. Leibniz the principle that is informally known as "substituting equals for equals" and is formally expressed by

x = y ⇒ f.x = f.y ,

which we can also read as "function application is equality-preserving". This note proves the

Theorem Besides the two constant relations, equality is the only relation preserved by function application.

In this note, function application is denoted by an infix dot (period); for the sake of simplicity we restrict ourselves to "endofunctions", i.e. functions for which argument and function value are of the same type, which is also the type of the variables (a, b, c, d, x, y) and the type on which the binary relations are defined. Boolean negation will be denoted by ¬ , boolean equality by ≡ , which is given the lowest syntactic binding power ; the dot of function application has the highest binding power.

For the rest of the note, R denotes – in the usual infix notation – a binary relation

• that differs from the constant relation T, given by 〈∀ x,y :: x T y〉, and

• that differs from the constant relation F, given by 〈∀ x,y :: ¬(x F y)〉, and

• that differs from equality.

To prove our theorem we have to show that, in general, R is not preserved by function application, i.e. that there exist an a,b and f such that

a R b and ¬(f.a R f.b) .

We take care of the existential quantification over f by using the lemma – not proved here –

〈∃ f :: f.a = c ∧ f.b = d〉  ≡   a = b ⇒ c = d ,

which allows us to rewrite our proof obligation as follows:

Show the existence of a quadruple a, b, c, d such that

(i) a R b
(ii) ¬(c R d)
(iii) a = b ⇒ c = d

We now show this existence by constructing a witness.

That R differs from equality enables us to postulate (for the rest of this note) for some x,y

x R y ≢ x = y

and now we distinguish two cases.

Case 0: x R y ∧ x ≠ y
Choosing a,b := x,y ensures that (i) and (iii) are satisfied; because R differs from T, c,d can be chosen so as to satisfy (ii). (End of Case 0)

Case 1: ¬(x R y) ∧ x = y
Choosing c,d := x,y ensures that (ii) and (iii) are satisfied; because R differs from F, a,b can be chosen so as to satisfy (i). (End of Case 1)

Since the case analysis was exhaustive, our proof obligation has been discharged.

Remark The unproved lemma can be demonstrated by mutual implication. The one direction – left ⇒ right – follows directly from Leibniz's Principle; the other direction – the one needed in this note – requires case analysis, say a=b versus a≠b. (End of Remark.)

Austin, 17 September 1996

prof. dr. Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188
USA

transcribed by Hasnain Mujtaba
revised 19-Jul-2019
