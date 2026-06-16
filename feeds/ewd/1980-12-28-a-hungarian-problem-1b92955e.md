---
title: "A Hungarian problem (EWD765)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/ewd765.html
published: "1980-12-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD765.PDF
---

# A Hungarian problem

Martin Rem returned from his trip to Hungary with the following problem. Let two positive integers p and q be given.
We start with a bag containing a finite number of integers. When the bag contains some number, x say, at least twice,
a move is possible; the move consists of replacing in the bag two occurrences of x by x+p and x−q respectively.
Show that this game terminates.
When we characterize the contents of the bag by its characteristic function f —i.e. for all x, f (x) = the number of
occurrences of the value of x in the bag— and use the initializing guard —which is true when the equation can be
solved and sets the unknown to a solution— the game can be written as follows

do x : f (x) ≥ 2 →

f (x−q), f (x), f (x+p) := f (x−q)+1, f (x)−2, f (x+p)+1

od

To start with, we generalize the game: for each move a new pair (p, q) of positive numbers may be chosen.
Furthermore we extend our program with auxiliary array n, initially satisfying for all x   f (x) = n(x),
and modified by extending the guarded command with the statement

n(x−q), n(x+p) := n(x−q) + 1, n(x+p) + 1

In other words, after initialization n(x) counts how many times f (x) is increased.
With c defined as c = (N x :: n(x) > 0) we have
Lemma 0.
Each n(x) is bounded or c is unbounded.
Proof. Since each f (x) is bounded by the constant number of integers in the bag, an f (x) that is increased an
unbounded number of times is also decreased an unbounded number of times, causing increases
of n(y) for some y > x (and some y < x). Hence we have for all x:

n(x) is bounded or (S y : y >x : n(y)) is unbounded.

Hence, for bounded c there is no largest x for which
n(x) is unbounded, hence there is no such x at all. (End of proof of Lemma 0).
Since “c is bounded and each n(x) is bounded” is equivalent with “the game terminates”, and since P ∨ Q equals
(¬Q ∧ P) ∨ Q, we derive from Lemma 0:
Lemma 1
The game terminates or c is unbounded.
The smallest value in the bag is the smallest value x such that n(x) > 0 ; the largest value in
the bag is the largest value x such that n(x) > 0 . Hence, from Lemma 1 we conclude
Lemma 2.
The game terminates or the difference between largest and smallest value in the bag is
unbounded.
With the derivation of Lemma 2, the auxiliary array n has done its work. Defining “a gap of length h” as a solution
of the equation
f (x) > 0 and f (x+h) > 0 and (A j : x < j < x+h : f (j)=0)
we conclude from Lemma 2, because the number of integers in the bag is constant,
Lemma 3.
The game terminates or the length of the largest gap is unbounded.
Finally we undo some of our generalization: let the choice of the pair of positive numbers for each move be
restricted to pairs (p, q) such that p+q ≤ K for some K. We then have
Lemma 4.
No gap exceeds the maximum of K and the length of the largest initial gap.
Combination of the last two lemmata solves the Hungarian problem.
Acknowledgement and concluding remark.
Thanks are due to the members of the Tuesday Afternoon Club with whom I discussed my first proof of Lemma 2.
That proof used mathematical induction on the number of integers in the bag. I am particularly grateful to
R.W. Bulterman for the persistence of his dislike of the three-case analysis in the induction step required;
He also objected to some handwaving with which I draw a correct but sweeping conclusion from “A possible move
remains possible until done”.
I must admit that I would have liked a simpler termination proof than the one I found this evening. Perhaps
it doesn’t exist: Hungary is a traditional source of non-trivial problems.
Considering first the generalized problem was an improvement; relaxing constraints could be a general
technique for doing away with reductiones ad absurdum.

Plantaanstraat 52 December 1980

5671 AL NUENENprof. dr. Edsger W. Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

10-Nov-2015

.
