---
title: "The solution to a cyclic relaxation problem (EWD386)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD386.html
published: "1973-08-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD386.PDF
---

# The solution to a cyclic relaxation problem

EWD 386

The solution to a cyclic relaxation problem.

The problem solved in this note arose in connection with the (just initiated) study of self-stabilizing systems.

Consider a circle and N points numbered from 0 through N-1, placed in an arbitrary order around the circumference. For "the adjustment of point nr. i" we consider the shortest clockwise path along the circumference from its predecessor —i.e. point nr. (i-1) mod N— to its successor —i.e. point nr. (i+1) mod N—; the new position of point nr. i will be halfway (i.e. the middle) of that path. In formula (taking the circumference of the circle as unit) with pred = x[(i-1) mod N] and succ = x[(i+1) mod N]

x[i]:= if pred ≤ succ then (pred+succ)/2 else ((pred+succ+1)/2) mod 1 fi.

If we start doing adjustments, will the system converge on a stable state?

This is not necessarily the case if we do the adjustments simultaneously, i.e. determine all new positions in terms of all the old ones, as is shown by the following examples.

N = 3

|

N = 4

t
0
1
2
|
t
0
1
2

x[0]
0
1/2
0
|
x[0]
0
3/4
0

x[1]
2/3
1/6
2/3
|
x[1]
0
1/4
0

x[2]
1/3
5/6
1/3
|
x[2]
1/2
1/4
1/2

|
x[3]
1/2
3/4
1/2

For both odd and even N we have an example that will oscillate with a period 2. If, however, we do the adjustments one at a time in a fair random order (i.e. without permanent neglect of certain points), then the system is bound to converge.

Consider, instead of the N points, the N clockwise paths leading from each point to its successor. After a point adjustment the two paths meeting at that point will both be less than 1/2 and no future adjustment can ever undo that! After adjustments all around the circle each path will be less than 1/2, and from that moment onwards each triangle "i-1, i, i+1" is a clockwise one. The total clockwise path from 0 to 1, from 1 to 2, ....., from N-1 to 0 will go around the circle a fixed number of times, m say (0 ≤ m ≤ N/2). No adjustment can anymore change the value of m, from now onwards we could even do simultaneous adjustments. The final state will satisfy for all i

(x[(i+1) mod N] - x[i]) mod 1 = m/N.

The system converges linearly (imagine successive points connected by spiral springs or rubber bands of equal length).
The above was written under the assumption that along the circumference we had the continuum at our disposal, i.e. that the fractions x, satisfying 0 ≤ x < 1 could be represented in arbitrary precision. Suppose now that we have to represent the fractions x as integer multiples of 1/p (where we may assume the integer p to satisfy
p >> N).

The extent to which the system converges seems to depend on how we round off when necessary, i.e. when the clockwise path from predecessor to successor turn out to be an odd multiple of 1/p. If we impose the rule, that rounding will always take place in the same cyclic direction (say "anticlockwise"), then the following will happen.

With m defined as in the continuous case, define q and r by

m*p = q*N + r with 0 ≤ r < N.

Of the paths p[i] leading from x[i] to its successor, (N - r) —the "short" paths— will be of length q/p and r paths —the "long" paths— will be of length (q + 1)/p. If p[i-1] is "long" and p[i] is "short", adjustment of point nr. i —with anticlockwise rounding— will have the effect that the predicates "short" and "long" have interchanged position. The short ones will be travelling anticlockwise through the cycle, simultaneously the longs ones will travel clockwise through the cycle.

The two types of paths travelling in opposite direction through the same cycle makes it clear that if m*p is an integer multiple of N, that then the system will converge to a completely stable situation.

NUENEN, 8th August 1973

dr.Edsger W.Dijkstra

transcribed by Tristram Brelstaff

revised Tue, 27 Mar 2007
