---
title: Random Testing Gets No Respect
url: https://blog.regehr.org/archives/382
published: "2011-02-12T21:45:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=382
---

# Random Testing Gets No Respect

A funny thing about random testing is that it gets little respect from most computer scientists. Here’s why:

- Random testing is simple, and peer reviewers hate simple solutions.
- Random testing does not lend itself to formalization and proofs, which peer reviewers love.
- Random testing provides no guarantees. Peer reviewers love guarantees — regardless of whether the they are actually useful.
- Random testing is hard to do well, but the difficulty is more art than science: it requires intuition, good taste, and domain knowledge. These — like many other important things — can’t be neatly packaged up in a journal paper.
- People dislike the probabilistic aspect, and would prefer a fixed test suite.
- Most researchers don’t produce robust software, and therefore have trouble understanding why anyone would need testing techniques that can help produce robust software by uncovering difficult corner-case bugs.

Of course, random testing has plenty of well-known problems, but I often get the feeling that people are responding to prejudices rather than facts. Even so, the picture is far from bleak and there are plenty of good researchers who exploit or at least appreciate random testing. My favorite recent example is found in this [paper by Fox and Myreen](http://www.cl.cam.ac.uk/~mom22/itp10-armv7.pdf), who are working on a formal model of the ARM instruction set architecture in the HOL4 theorem prover. This kind of work is the most pedantic, hard-nosed possible kind of formal methods, and yet they validate their model (and find bugs in it) using humble randomized differential testing — against real ARM boards, no less.
