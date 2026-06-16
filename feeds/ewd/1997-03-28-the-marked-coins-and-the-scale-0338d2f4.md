---
title: "The marked coins and the scale (EWD1260)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1260.html
published: "1997-03-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1260.PDF
---

# The marked coins and the scale

EWD 1260

The marked coins and the scale

Our tool is a simple scale with two plates, as is used by the goddess Justitia; a weighing has one of 3 possible outcomes: the scale balances, or it tips in the one direction, or it tips in the other direction.

All genuine coins have the same weight, a fake coin is heavier or lighter. We are presented with a number of “marked coins” of which we know:

•  exactly 1 of the marked coins is fake,

•  a coin marked as “pluscoin” is genuine or too heavy,

•  a coin marked as a “minuscoin” is genuine or too light.

We are considering the problem of identifying the fake coin in n weighings. Because a sequence of n weighings has 3n outcomes, one cannot guarantee the identification of the fake one among more than 3n coins to be investigated, but this upper bound can be achieved. By mathematical induction, we shall prove that n weighings suffice to identify the fake one among 3n marked coins. At the same time we shall derive all protocols achieving this identification.

Because 30=1, the base (i.e., n=0) is trivial: in the case of 1 marked coin, that one is the fake one and 0 weighings suffice.

For the induction step, we write the number of marked coins among which the fake one is to be identified as

(0)     P + M + p + m + t    ,

where at the first weighing

P = # pluscoins on the left-hand plate

M = # minuscoins on the left-hand plate

p = # pluscoins on the right-hand plate

m = # minuscoins on the right-hand plate

t = # coins left on the table

In order to enable the first weighing to give enough information to localize the fake coin, we insist that both plates contain the same number of coins, i.e.

(1)     P + M = p + m    .

Under that constraint, tipping or balancing of the scale can be related to the place of the fake coin, more precisely:

•  If the left-hand plate goes down, the fake coin is either among the P pluscoins or the m minuscoins, i.e. within a set of P+m coins.

•  If the left-hand plate goes up, the fake coin is either among the M minuscoins or among the p pluscoins, i.e. within a set of M+p coins.

•  If the scale balances, the fake coin is among the t coins on the table.

To maximize (0), we independently maximize P+m, M+p, and t. Since we have still n weighings left, this yields ex hypothese

(2)
P +m = 3n

M+p = 3n

t = 3n

which yields that (0) equals 3n+1 as desired, provided (1) and (2) have a solution. Elementary algebra suffices to show that they are equivalent to

(3)
P = p

M = m

P+M = 3n

t = 3n

which tells us that in the 1st weighing, both sides should contain equal numbers of pluscoins and equal numbers of minuscoins.

Such a solution indeed exists. Start with the plates empty and all 3n+1 coins on the table. A move consists of taking either two pluscoins or two minuscoins from the table and putting one of those on each plate, until (# coins left on the table) = 3n. Because the number of coins on the table is odd, it is ≥ 3 when a move still has to be performed: with 2 markings, and ≥ 3 coins, at least 2 coins have equal marking.

The solution of (3) is not necessarily unique: for instance, with n=2 and the marking +++++----, we have the following two solutions

left
right
table

++-
++-
+--

+--
+--
+++

*                        *

*

Equation (1) can be dropped if we admit (fake) coins of negative weight and have our genuine coins being weightless. The same freedom is obtained by the availability of enough genuine coins: if P+M ≠ p+m, some genuine coins can then be used to make up for the difference, so that both plates carry the same number of coins. (This was the situation that prevailed in EWD1083.)

I gave the problem of EWD1083 (unmarked coins with at most one fake coin, and as many genuine coins as you like) to my students. Almost all of them found solutions, sometimes of considerable ingenuity. The ingenuity was always involved in the creation of a very special solution. The purpose of this note is to draw attention to the fact that finding the most general solution could very well be the simplest task. In the above I did not feel to have to invent anything.

Austin, 31 March 1997

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Xiaozhou (Steve) Li

revised Mon, 10 Dec 2007
