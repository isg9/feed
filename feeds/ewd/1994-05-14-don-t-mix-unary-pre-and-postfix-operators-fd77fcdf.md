---
title: "Don’t mix unary pre- and postfix operators (EWD1181)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1181.html
published: "1994-05-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1181.PDF
---

# Don’t mix unary pre- and postfix operators

Don't mix unary pre- and postfix operators

One of the operators of the relational calculus is the negation (or "complement") which I denote by a prefix  ¬ . Another unary operator is the transpose (or "converse"), which I denote by a prefix ~ . The two operators commute, so one never knows whether to write ¬~X or ~¬X . It has been suggested to eliminate this dilemma by making one of the two, say ~ , a postfix operator. This allows us to write ¬X~ , leaving open whether this is to be parsed (¬X)~ or ¬(X~).

Commuting operators at different sides of the operand! ("A neat idea!" as Ollie North would say.) But you run into difficulties with the introduction of the first unary operator that commutes with neither. It fits at neither side and precedence rules or parentheses are needed for disambiguation. With all unary operators at the same side, There would have been no problem at all. Moral: Beware of neat ideas.

Winding Stair Campground

14 May 1994

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-118, USA

transcribed by Tristram Brelstaff

revised Sun, 30 Dec 2007
