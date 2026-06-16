---
title: "The problem of the Swiss football players (EWD685)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD685.html
published: "1978-11-16T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD685.PDF
---

# The problem of the Swiss football players

EWD685-0

The problem of the Swiss football players.

Given a finite directed graph that is strongly connected, i.e. such that a directed path exists from any node to any other. At each node a Swiss football player is placed and one has the ball. The football player who has the ball can kick it along any of its outgoing edges; being Swiss, the players are fair, i.e. no player will permanently refuse to kick the ball to one of the players he could kick it to. (More precisely: when the number of times a player receives the ball is unbounded, so is the number of times he has kicked it along any of his outgoing edges.) Show that, as the game continues, each player receives the ball an unbounded number of times.

I have asked a number of mathematicians to prove the above (trivial) theorem. Essentially I got two proofs.

Proof A. Assume that one player gets the ball only a finite number of times. Then we can conclude that all players who could kick the ball to him, being Swiss, have also received the ball only a finite number of times. By induction, the graph being strongly connected, we can conclude that all players get the ball only a finite number of times. Because the graph is finite, this would imply that the ball is kicked a finite number of times: on the other hand the game can go on forever. Contradiction!

Proof B. Because the game can go on forever and there is only a finite number of players, there is at least one player who gets the ball an unbounded number of times; this player, being Swiss, therefore kicks the ball an unbounded number of times to each player he can kick it to, who, therefore also receives the ball an unbounded number of times. The graph being strongly connected, we conclude by induction that each player receives the ball an unbounded number of times.

The proofs are very similar, both rely on an induction argument that is sound for strongly connected graphs. Proof A, however, is phrased as a reductio ad absurdum. Proof B is formulated as a direct proof.

The startling observation was that all computing scientists, whom I asked to prove this simple theorem, essentially gave Proof B, whereas all other mathematicians I asked how they would prove it, gave essentially Proof A! My sample was small: 3 computing scientists and 6 other mathematicians. The sample was too small to make the experiment very significant; moreover the 3 computing scientists know me much better than the 6 other mathematicians. Yet I found the outcome striking enough to record it: I –perhaps wishfully– interpret it as a confirmation that in computing science indeed something is happening.

Remark. Two of the 6 gave as first reaction "I would try to reduce it to a known theorem." (End of remark.)

Plataanstraat 5

5671 AL NUENEN

The Netherlands
prof.dr.Edsger W.Dijkstra

BURROUGHS Research Fellow

transcribed by Alejandro Ruiz

revised Wed, 14 Nov 2007
