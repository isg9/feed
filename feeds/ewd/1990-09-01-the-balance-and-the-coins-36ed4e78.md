---
title: "The balance and the coins (EWD1083)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1083.html
published: "1990-09-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1083.PDF
---

# The balance and the coins

EWD 1083

The balance and the coins

All true coins have the same weight, a false coin is too heavy or too
light. Given (3n-1)/2 coins of which at most 1 coin is
false, we have to determine whether they are all true, and, if not, we
have to identify the false coin and to determine whether it is too
heavy or too light. Our equipment consists of an additional coin
known to be true and a balance that may be used for n weighings.

Let us denote (3n-1)/2 by s.n. Then s.(n+1) = 3∙s.n + 1. We show how to solve the problem for s.(n+1) coins in n+1 weighings in terms of a solution of n weighings for s.n coins.Our first weighing involves 2∙s.n + 1 of the coins to be investigated. With the balance we compare s.n+1 of those with the other s.n, complemented with the coin known to be true.
If the two sets are of equal weight, we use the remaining n weighings to solve the problem for the s.n coins that have not been on the balance yet.

If the two sets are of different weight, we have 2∙s.n + 1 suspect coins, more precisely, s.n+1 suspects of the one polarity and s.n suspects of the other polarity. We make s.n “double coins” by pairing suspects of different polarity; this leaves one suspect of known polarity unpaired. We now use the remaining n weighings to solve the problem for the s.n “double coins”. If all double coins are “true”, the unpaired suspect (whose polarity is known) is the false coin. If one of the double coins is “false”, the polarity of its falseness determines which of its two constituent suspects is the false coin.

Note For n>0, s.n ≥ 1. In that case we have enough —i.
e. 2— coins known to be true to make a true “double coin” when we need it. (End of Note.)

The above takes care of the induction step. For the base it suffices
to observe that s.0=0 and that for 0 coins, 0 weighings suffice to
determine that all the coins are true.

*                       *

*

Variations of this problem circulated in the United Kingdom during WWII, and after the war in the rest of Europe — e.g. 12 coins, one of which is false, 3 weighings, but no 13th coin known to be true. It surfaced this week in the department and turned out to be new for my graduate students. The reason that I took the trouble of writing down the above, is that it provides a nice example of a forced argument.

With s.n possibilities for the false coin, 2 possibilities for the polarity of its falseness, + the possibility of all coins true, our weighing algorithm has to distinguish between 2∙s.n + 1 cases. But, 2∙s.n + 1 being equal to 3n, and a weighing having 3 possible outcomes, the only way in which an algorithm of n weighings can give rise to 3n different computations is if for each weighing, all 3 outcomes are possible. There is no slack.

In other words: faced with s.(n+1) coins, for which we have to distinguish between 3n+1 cases, each outcome of the first weighing has to reduce the number of possibilities to 3n, no more and no less. Hence the number of coins not involved in the first weighing (which can imply: no false coin on the balance) has to be s.n, no more and no less. Because s.(n+1)-s.n —being 2∙s.n + 1— is odd, the need for a coin known to be true follows. The introduction of the s.n “double coins” is undeniably an invention, but I claim that it is only a minor one: if we wish to keep things simple, we should reduce the s.(n+1)-problem to the s.n-problem, irrespective of the outcome of the first weighing. In the one case —equal weights— we get this reduction for free, in the other case —different weights— we have to construct the reduction. I call the invention “minor” —“minuscule” is perhaps better— because I cannot think of any other way of constructing this reduction. The hallmark of the experienced avoider of avoidable complications is probably the immediate awareness that, also in the case of different weights in the first weighing, we should leave ourselves with an s.n-problem. In more than one sense the construction of the above solution has been a balancing act.

Austin, 1 September 1990

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Xiaozhou (Steve) Li

revised Mon, 28 Jan 2008
