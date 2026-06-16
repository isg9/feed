---
title: "The undeserved status of the pigeon-hole principle (Mathematical Methodology) (EWD1094)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1094.html
published: "1991-03-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1094.PDF
---

# The undeserved status of the pigeon-hole principle (Mathematical Methodology)

EWD 1094

The undeserved status of the pigeon-hole principle

Right at the moment I was introduced to the pigeon-hole principle, I understood that there was something very special about it, for the Englishman from whom I learned it told it to me under its German name "das Schubfach prinzip". Later I learned that also in other languages it is honoured by a special name. It is, indeed, surrounded by some mystique, for proofs using it are often regarded as something special, something particularly ingenious. This feeling of awe is, for instance, nicely reflected by Ross Honsberger when he concludes one of his examples with the characterization "another triumph of the pigeon-hole principle". (Earlier he has called it "a fundamental tool of combinatorics".)

It is my feeling, however, that this mystique and the surprise when we see the pigeon-hole principle being applied successfully are closely connected to the clumsiness of the usual formulations of the principle: they contain a lot of misleading noise, are overspecific, and hide the principle's arithmetic nature. Instead of composing an average formulation, I quote a typical one verbatim from the literature:

(0)
"if more than n objects are distributed into a set of n compartments, some compartment must receive more than one of the objects".

(Instead of the above "more than n", one often encounters the more specific "n+1".) We shall compare (0) with the alternative

(1)
"for a nonempty, finite bag of real numbers, the maximum is at least the average (and the minimum is at most the average).

Comparing (1) and (0), we observe to begin with that the "distributing" and "receiving" have disappeared, and rightly so, for they are just noise: the pigeon-hole principle is equally applicable when no process of distribution and reception is involved, e.g. when we wish to conclude, from the fact that a month is longer than a week, that in a month some weekday occurs more than once.

The second thing we observe is that the "objects" and the "compartments" have disappeared. That is a great improvement, for it saves us the needless trouble of making a painful identification in all those circumstances in which (for visual or linguistic reasons) the metaphor of objects and compartments does not fit nicely. In the previous example, for instance, we should not talk about a month containing a number of Tuesdays, but about Tuesday ( = compartment ) containing a number of days-of the-month ( = objects ). The whole metaphor of objects and compartments is just a pain in the neck.

Another regrettable consequence of the metaphor is that it invites a misleading visualization. Formulation (0) is in the form of an implication; the contrapositive states equivalently

(2)
"if each of n compartments contains at most one object, they contain together at most n objects".

Formulation (0) evokes a picture of cramped, overstuffed compartments —2 or more pigeons squeezed into a 1-pigeon hole— , whereas formulation (2) rather evokes a picture of underoccupied compartments. The difference between the pictures hides the fact that (0) and (2) are equivalent.

Remark It is not uncommon to see people distinguish —not in meaning, but in rôle— between x≤y and y≥x . The former they read as expressing (in terms of y) an upper bound for x , the latter as giving (in terms of x) a lower bound for y . I advise against trying to make such distinctions: the charm of

average ≤ maximum                  and

maximum ≥ average

is that they state exactly the same and need not evoke different connotations as (0) and (2) seem to do. (End of Remark.)

Finally, the traditional pigeon-hole principle is much less general than it should be: expressed in average and maximum, (0) yields

average > 1 ⇒ maximum >1

which is much weaker than (1)'s

average ≤ maximum ;

this is why (1), being really stronger than (0), is referred to as the Generalized Pigeon-hole Principle.

As an example in which the Generalized Pigeon-hole Principle comes in handy, we consider now what is known as the Problem of the German Soccer Lotto.

Variables of type "outcome" have 1 of 3 possible values; variables of type "column" consist of 13 variables of type "outcome". In the following, dummies c and d range over type "column" and W is a set of constants of type "column". We are asked to construct a set W, as small as possible, such that

(A d :: (E c : c ∈ W : (N i : 0 ≤ i < 13 : d.i = c.i) ≥ 5))

In words: W should be such a set of columns that, whatever column we choose for d, W contains a column c that coincides with d in at least 5 positions.

Set W contains at least 3 columns, for with at most 2 columns in W it is possible to choose d such that it coincides in no position with a column in W. But 3 columns suffice. Let x, y, z be the three columns in W; we can choose them such that for any i in the range 0 ≤ i < 13

{ x.i, y.i, z.i }

is the triple of possible outcome-values. With such a choice, each value d.i coincides with one of the three { x, y, z }; 13 coincidences in 3 columns yields an average of 4⅓ coincidences per column, and hence at least 1 column coincides in at least 5 positions with d . And thus the Problem of the German Soccer Lotto has been solved.

The example has been included for two reasons. Firstly it is an example for which the traditional (0) does not suffice and the stronger (1) is really needed. Secondly, it shows the arithmetic reason for the surprise that the use of the pigeon-hole principle often evokes: the three constants 3, 13 and 5 of the problem statement fortunately satisfy the magical relation that 5 is the smallest integer that is at least 13/3. It is this arithmetic accident that makes the Problem of the German Soccer Lotto trivial. Replacing 5 by 6 we get a completely different problem, which is much harder to solve! Another way to phrase the same observation is that if you can get away with an argument as simple as the pigeon-hole principle, you have just been lucky.

*                    *

*

Another, I think unpleasant, property of the pigeon-hole principle is that it seems to invite the reductio ad absurdum. Take this simple problem:

On the ranch, each cowboy is the exclusive owner of at least one horse. Show that on the ranch

# cowboys ≤ # horses   .

It is not unusual at all to encounter an argument of the following shape. Assume that the conclusion is false, i.e., assume that # cowboys > # horses; now put each cowboy (= pigeon) into one of the horses (= holes) he owns; then the pigeon-hole principle tells us that there is a horse owned by at least two cowboys, which contradicts the exclusive ownership. Hence the conclusion is true, i.e. # cowboys ≤ # horses.

[Compare the argument with the following one. Since the minimum number of horses per (i.e. exclusively owned by a) cowboy is at least 1, the average number of horses per cowboy is at least 1, i.e.

# horses

# cowboys

≥  1               .]

The former argument is hardly a parody. In a recent textbook on discrete mathematics, I encountered the following problem

"There are 42 students who are to share 12 computers. Each student uses exactly one computer and no computer is used by more than six students. Show that at least five computers are used by three or more students."

Let k be the number of computers serving at least three students; since each computer serves at most 6 students, these k machines serve at most 6k students. Since the remaining 12–k serve at most 2 students each, the latter machines serve at most 24–2k
students. Adding these results, we conclude that the number of students served is at most 24 + 4k. Since 42 students are served, we conclude 42 ≤ 24 + 4k or 4½ ≤ k and, since k is integer, 5 ≤ k , which completes the proof.

The textbook, however, proceeds as follows —ignore that, instead of 5≤k, it only shows k ≠ 4— :

"[Use an argument by contradiction.] Suppose that only four computers were used by three or more students. [ A contradiction must be derived]. At most six students are allowed to share any computer, making a total of at most 24 students using these four computers. Since there are 42 students in all, that would leave at least 18 students to share the remaining eight computers. But the generalized pigeon-hole principle guarantees that if 18 students share 8 computers, then at least three must share one of them. [The pigeons are the 18 students, the pigeon-holes are the eight computers, and 18 > 2∙8.] This is a contradiction. Thus the supposition is false, and so at least five computers are used by three or more students."

Remark Both arguments contain the number 24, but this is an accident: in the former argument, 24 emerges as twice the number of machines, in the latter as four times the maximum number of students served by a single machine. (End of Remark.)

In the problem statement, it is inessential that each student uses "exactly" one computer: with each student using "at least" one computer, the conclusion would have been valid as well. Is it far-fetched to assume that the overspecific "exactly" sneaked into the problem statement because a pigeon can occupy only 1 pigeon-hole at a time? We have now spotted (i) a needlessly weak theorem to be proved, (ii) an avoidable reductio ad absurdum, and (iii) an error in the purported proof (for k≥5 does not follow from k≠4). I am afraid that the desire to use the pigeon-hole principle can be blamed for all three shortcomings. The argument in terms of k is just an order of magnitude cleaner than the solution of the textbook.

The moral of the story is that we are much better off with the neutral, general standard procedure: name the unknown(s) so that you can formalize all conditions given or to be established and simplify by formula manipulation, unencumbered by imagery as tends to be imposed upon you by catchy names like "The Pigeon-hole Principle".

Austin, 21 March 1991

prof.dr.Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

_____________

Mathematical Methodology

transcribed by Mario Béland

revised Fri, 18 Jan 2008
