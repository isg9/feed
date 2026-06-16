---
title: "Pruning the search tree (EWD1255)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1255.html
published: "1997-01-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1255.PDF
---

# Pruning the search tree

This note deals with three classical, very similar problems: in a sequence of moves, a bunch of entities have to be transferred across a river. The problem is that the available boat is so small that a number of trips are needed, and that there are curious constraints as to which combinations of entities may occur separated from the rest. For each problem, the challenge os to find an acceptable schedule.

For all problems we adopt the rule of "no cancellation", i.e. we reject any schedule in which a subsequence of successive moves results in no change. Consequently, only a finite number of finite schedules need to be investigated for acceptability, and in that sense our little puzzles are trivial. They become, however, little mathematical problems if we accept mathematics as the art and science of effective reasoning and try to reduce the number of schedules to be inspected. This note is being written because we can use these little problems to illustrate various techniques.

In all our problems, the river runs north-south and has to be crossed from the west bank to the east bank.

*               *

*

In our first problem, the river has to be crossed by a shepherd with a wolf, a goat, and a cabbage. The shepherd is the only one that can operate the boat, which is so small that each time he can take at most one of the three with him across the river. The compatibility constraint is that, in the absence of the shepherd. the wolf would eat the goat, and the goat would eat the cabbage. The challenge is to design (if possible) a schedule in which everything reaches uneaten the east bank.

With an arrow from A to B representing "A eats B", we can represent the incompatibilities by the picturebut this represents more than we are interested in: the important thing is that the shepherd does not leave wolf and goat together and unattended, regardless of whether the wolf could eat the goat or the goat could eat the wolf. The asymmetry of "A eats B" and hence the direction of the arrows is a red herring, and we can simplify the picture thus:

But now the constraints are symmetric between wolf and cabbage, i.e. also the difference between wolf and cabbage is "noise" and we can regard them as different instances of say, "alphas".

So, in our simplified problem, the shepherd arrives with one goat and two alphas, and the compatibility constraint is that, in the absence of the shepherd, no alpha may be in the company of the goat. The reader is invited to check that, with "no cancellation" as well, there is each time exactly 1 thing the shepherd can do:

(0) eastward with the goat

(1) westward alone

(2) eastward with an alpha

(3) westward with the goat

(4) eastward with an alpha

(5) westward alone

(6) eastward with the goat

Our solution is symmetric around move (3). Please note that at this level of abstraction —in which the two alphas are as indistinguishable as the two electrons in the helium atom— the solution is unique. Not distinguishing between the alphas is another way of achieving the efficiency of the counting argument. Because in this simple problem we only count to two, the abstraction has only halved our search space.

*               *

*

[In view of EWD1250, "The couples, the river, and the little boat", the inclusion here of the following problem exposes me to the accusation of typically Texan crime of self-plagiarism. Ora pro nobis.] Here is how Morris Kline presented the problem in 1967: "Two husbands and two wives have to cross a river in a boat which can hold only two people. How can they cross so that no woman is in the company of a man unless her husband is also present?"

We can name the four people by considering the couples (H,W) and (h,w) and the enumerate the disallowed combinations:

wH   wHW   Wh   Whw

In this terminology we can explore the tree of all possible scenarios, but that is an extensive search.

The problem is symmetric in the two couples and naming them differently ,as we did above, is of course sowing the seed of a combinatorial explosion. (Note that we had to mention "wH" and "Wh" separately!)

When we wish to ignore the distinction between the two couples, i.e. between upper and lower case in our enumeration of disallowed combinations, there is a little else but count the number of wives and husbands: they contain one or two wives and exactly one husband. Focusing on the latter, the disallowed combinations are ruled out if we impose the stronger constraint:

the husbands stay together  .

But as soon as the husbands stay together, we don't need to distinguish between them —they could be truly identical twins—, nor between the wives. The pairing, as we represent by the couples, has disappeared from the picture, and after this abstraction, there is only one solution left:

(0) westward two wives

(1) eastward one wife

(2) westward two husbands

(3) eastward one wife

(4) eastward two wives

The reader is invited to check that, together with the rule of "no cancellation", the above solution is indeed unique. Our gain is considerable: in terms of the couples (W,H) and (w,h), the original problem admits 8 different solutions.

Two remarks are in order. The minor remark is that the rule of "no cancellation" has been strengthened by the decision not to distinguish between the two wives. Had we maintained the distinction between the two wives, the rule of "no cancellation" would not have ruled out the continuation

(3) eastward one wife

(4) westward the other wife

The more general remark concerns our adoption of the stronger constraint "the husbands stay together". Strengthening the constraints is a perfectly general way of pruning the search tree, but one runs, of course, the risk of choosing the constraints so strong that they can no longer be met. It may be worth noticing that in this example the major effect of the stronger constraint was not so much a direct size reduction of the search tree but rather the introduction of the freedom to eliminate the distinctions between the husbands and between the wives. So much for the problem of the couples, the river, and the little boat.

*               *

*

Our third problem is very well known. "Three missionaries and three cannibals have to cross a river in a boat which can hold only two people. As soon as, at either bank, there would be more missionaries than cannibals, the missionaries would convert the cannibals. How can cross the river so that no cannibal conversions take place?"

The only way in which I could solve this problem, was by formulating some simple theorems, such as

(i) the permissible divisions (1,5) are those in which the single person is a missionary.

(ii) the permissible divisions (2,4) are those in which the pair contains at least 1 missionary.

(iii) the permissible divisions (3,3) are those in which each triple consists of missionaries only or of cannibals only.

In the following solution, m stands for a missionary, c stands for a cannibal, while x and y stand for either. The reader is, again, invited to check that at each step we have no choice: the solution below is unique. (It is only if you feel obliged to choose an x and a y, that you get 4 solutions.)

(0) → m x

(1) ← x

(2) → m m

(3) ← m

(4) → c c

(5) ← m c

(6) → c c

(7) ← m

(8) → m m

(9) ← y

(10) → m y

The above solution is as symmetric around its midway move as the two previous solutions.

*               *

*

The counting abstraction

In the first problem, we eliminated the distinction between the wolf and the cabbage by the introduction of the alphas, and the solution became unique.

In the second problem we strengthened the constraints, so that we could eliminate the distinction between the husbands and, more importantly, the distinctions between the wives, and, as a result, the solution became unique.

Just maintaining the distinction between the cannibals on the one hand and the missionaries on the other hand yields 4 different solutions. Of course we could distinguish between the cannibals by calling them Ping, Peng and Pong respectively. (This is analogous to distinguishing between the two alphas by calling them "wolf" and "cabbage" respectively.) Similarly we could distinguish between the missionaries by calling them Mark, Luke, and John respectively. I would like the reader to realize that, salvo errore, the number of solutions in terms of named cannibals and named missionaries equals 8100. Compare this number with the original 4 and you see how large the price can be that has to be paid for the introduction of needless distinctions.

It is this observation that should give us a fresh appreciation of the effectiveness of counting arguments. In the traditional reconstruction of Mankind's discovery of the number concept, Man is supposed to observe that "five apples, take away one apple, remain four apples", then "five pears, take away one pear, remain four pears", and then with nuts, pebbles, etc. until the generalization emerges "five, take away one, remain four" or "5-1=4". I remember those reconstructions stressing the abstraction from the difference between apples, pears, nuts, pebbles, etc.

But there is a more profound abstraction going on: there is a profound difference between "the number five" on the one hand and "five things of unspecified nature" on the other. In the case of "five things, take away one thing", it is appropriate to ask for further instructions: "Which of the five things shall I take away?", but when asked to calculate "5-1", the question "Which 1 shall I subtract?" is meaningless.

The essence of counting is not distinguishing between the objects counted, and, conversely, if we have a collection of indistinguishable objects, their number seems to be about the only thing that can matter.

This is nice to know.

Remark I cannot resist the temptation to draw the attention to what now almost strikes me as an anomaly, viz. that in all "fundamental" considerations about "number" and "counting" that I remember, the objects counted are most emphatically distinguishable.

For instance, in 1884, F.L.L. Frege (1848-1925) defined the cardinal number of a given class as the class of all those classes similar to the given class. But when the notion of class was made precise in 1895 by G. Cantor (1845-1918), he defined it thus [translation published by E.T.Bell]: "By a class (Menge) we understand any summary (Zusammenfassung) into a single whole of well-distinguished objects of our intuition (Anschauung) or of our thought (Denken)."

Similarly, counting is always described as counting the objects "in a given order" but if the objects are indistinguishable there is no way of "giving" the order. The order is essential for the counting process, not for the latter's outcome. (End of Remark.)

I gratefully acknowledge my discussion on most of the above with Rutger M. Dijkstra and Silvija Seres.

Austin, 19 January 1997

Prof. Dr. Edsger W. Dijkstra

Department of Computer Science

The University of Texas at Austin

Austin, TX 78712-1188 USA

Transcription by Jorge Andrés Pérez.

Last revised Mon, 10 Dec 2007.
