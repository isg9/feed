---
title: "An unfortunate use of symbols (EWD1206)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1206.html
published: "1995-05-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1206.PDF
---

# An unfortunate use of symbols

I quote from a recent note “A Graphical Calculus” by Sharon Curtis and Gavin Lowe [Oxford University Computing Laboratory, Parks Road, Oxford, OX1 3QD, UK], just to bring you in the picture:

Formally, we consider graphs of the form (V, s, t, A) where V is a finite set of vertices, s∈V is the source, t∈V is the target, and A∈P(V×S×V) is a finite set of edges labelled with elements of S representing relations: the edge (v, R, v') represents an edge from v to v' labelled R.

So far, so good (that is, as long as the use of the verb “to represent” does not bother you). But half a dozen lines further they continue

We can now formally define the way in which a graph represents a relation.

Definition 1  The graph G = ({v0,..., vn}, v0, vn, A) represents the relation R where

(0) xRy iff (∃x0,...,xn • x = x0 ∧ y = xn ∧
∀(vi, S, vj)∈A • xi S Xj).

We call R the interpretation of G."

Remark  In the printed note from which I quote, (0) occupied a single line and the outer parenthesis pair was lacking. (End of Remark.)
There are all sorts of things wrong with “formula” (0). Minor things are that the use of x in the same context with and without subscript can be viewed as a “clash of identifiers” and that the use of the ellipse “,...,” almost certainly takes this character sequence outside the range of well-defined formulae. Much worse is the crummy use of quantification. Given an expression, one can quantify over one of its global (= free) variables, which then becomes a dummy, i.e. a local (= bound) variable of the quantification, in exactly the same way as in which lambda abstraction turns a global variable into a local one. I know how to deal with dummy variables but there are no such things as dummy expressions.

This immediately disqualifies the existential quantification as soon as we decide to view subscription as a notational alternative to function application (which is the only way in which I can cope with subscription). The universal quantification suffers from the added complication that the syntactic status of i and j is a complete mystery.

It was Rutger M. Dijkstra who, at the ATAC session of 25 April 1995, pointed out that the source of the trouble is the definition of G with its numbered vertices, which then calls for the numbered x-values. He eliminated those two functions on numbers (and the numbers themselves) ℕ→V and ℕ→X by introducing the (dummy) f : V → X :

Definition 1  The graph G = (V, s, t, A) represents the relation R where

xRy iff 〈∃f :: x = f.s ∧ y = f.t ∧〈∀v, S, w : (v, S, w) ∈ A : (f.v) S (f.w)〉〉.

By their poor use of symbols, the authors unnecessarily enfeeble their plea for their graphical calculus, but we can be grateful to them for reminding us once more of the penalty that subscription tends to entail.

Austin, 14 May 1995

prof. dr. Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712–1188
USA

transcribed by Swarup Sahoo
last revised Tue, 7 Jun 2011
