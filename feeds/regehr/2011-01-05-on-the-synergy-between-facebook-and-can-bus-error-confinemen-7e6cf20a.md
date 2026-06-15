---
title: On the Synergy Between Facebook and CAN Bus Error Confinement
url: https://blog.regehr.org/archives/335
published: "2011-01-05T04:44:33Z"
feed: regehr
guid: http://blog.regehr.org/?p=335
---

# On the Synergy Between Facebook and CAN Bus Error Confinement

One of my favorite lectures in my embedded systems class covers the design of CAN bus: a highly robust network most often used in automotive applications. CAN’s design is elegant in many ways and it includes several mechanisms to help keep the network operating under adverse conditions. One of these mechanisms is *error confinement* (see pp. 61-63 of the [CAN 2.0 standard](http://esd.cs.ucr.edu/webres/can20.pdf)). Each CAN node maintains a *transmit error count* and a *receive error count*. These are incremented when various transmit and receive errors are detected, and are decremented when the bus is used successfully. The idea is to build up a crude estimate of how faulty a node is. When either count exceeds 127 a node loses the right to transmit, and when either count exceeds 255 the node is required to remove itself from the network completely.

The other day I realized that I deal with Facebook friends in a similar way to how CAN bus deals with faulty nodes: if someone’s signal to noise ratio falls below some critical value I hide their subsequent updates. Previously I hadn’t formalized the rules but it’s something like this:

- Each friend’s score starts at 0
- +1 for any unoriginal political comment
- +3 for a particularly asinine political statement
- +1 for a remark about the performance of a college or professional sports team
- +1 for a cryptic personal comment
- +1 for describing the contents of any meal
- +1 for too much information
- Reset to 0 anytime the friend posts something offbeat, funny, original, relevant, or interesting
- Hide anyone whose count reaches 6

So far I’ve hidden just 5 friends out of 161 — not too bad. I suspect the number would be a lot higher if those appallingly stupid Farmville updates couldn’t be blocked separately.
