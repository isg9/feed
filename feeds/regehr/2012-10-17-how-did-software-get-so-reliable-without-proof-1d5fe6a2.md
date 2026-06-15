---
title: How Did Software Get So Reliable Without Proof?
url: https://blog.regehr.org/archives/820
published: "2012-10-17T16:26:47Z"
feed: regehr
guid: http://blog.regehr.org/?p=820
---

# How Did Software Get So Reliable Without Proof?

Tony Hoare’s 1996 paper [How Did Software Get So Reliable Without Proof?](https://www.gwern.net/docs/math/1996-hoare.pdf) addresses the apparent paradox where software more or less works without having been either proved correct or tested on more than an infinitesimal fraction of its execution paths. Early in the paper we read:

> Dire warnings have been issued of the dangers of safety-critical software controlling health equipment, aircraft, weapons systems and industrial processes, including nuclear power stations. The arguments were sufficiently persuasive to trigger a significant research effort devoted to the problem of program correctness. A proportion of this research was based on the ideal of certainty achieved by mathematical proof.
>
> Fortunately, the problem of program correctness has turned out to be far less serious than predicted. A recent analysis by Mackenzie has shown that of several thousand deaths so far reliably attributed to dependence on computers, only ten or so can be explained by errors in the software: most of these were due to a couple of instances of incorrect dosage calculations in the treatment of cancer by radiation. Similarly predictions of collapse of software due to size have been falsified by continuous operation of real-time software systems now measured in tens of millions of lines of code, and subjected to thousands of updates per year.

So what happened? Hoare’s thesis—which is good reading, but largely uncontroversial—is that a combination of factors was responsible. We got better at managing software projects. When testing is done right, it is almost unreasonably effective at uncovering defects that matter. Via debugging, we eliminate the bugs that bite us often, leaving behind an ocean of defects that either don’t manifest often or don’t matter very much when they do. Software is over-engineered, for example by redundant checks for exceptional conditions. Finally, the benefits of type checking and structured programming became widely understood during the decades leading up to 1996.

The rest of this piece looks at what has changed in the 16 years since Hoare’s paper was published. Perhaps most importantly, computers continue to mostly work. I have no idea if software is better or worse now than it was in 1996, but most of us can use a computer all day without major hassles. As far as I know, the number of deaths reliably attributed to software is still not hugely larger than ten.

Software, especially in critical systems, has grown dramatically larger since 1996: by a [factor of at least 100 in some cases](http://www.inf.ufpr.br/lmperes/esw_computer.pdf). Since the factor of growth was probably similar during the 1980-1996 period, it seems a bit odd that Hoare didn’t include much discussion of modularity; this would be at the top of my list of reasons why large software can work. The total size of a well-modularized software system is almost irrelevant to most developers, whereas even a small system can become intractable to develop if it is sufficiently badly modularized. My understanding is that both the Windows NT and Linux kernels experienced serious growing pains at various times when coupling between subsystems made development and debugging more difficult than it needed to be. In both cases, a significant amount of effort had to be explicitly devoted to cleaning up interfaces between subsystems. When I attended the GCC Summit in 2010, similar discussions were underway about how to slowly ratchet up the degree of modularity. I’m guessing that every large project faces the same issues.

A major change since 1996 is that security has emerged as an Achilles’ Heel of software systems. We are impressively poor at creating networked systems (and even non-networked ones) that are even minimally resistant to intrusion. There does not appear to be any evidence that critical software is more secure than regular Internet-facing software, though it does enjoy some security through obscurity. We’re stuck with a deploy-and-patch model, and the market for exploits that goes along with it, and as far as I know, nobody has a better way out than full formal verification of security properties (some of which will be very difficult to formalize and verify).

Finally, formal correctness proofs have not been adopted (at all) as a mainstream way to increase the quality of software. It remains almost unbelievably difficult to produce proofs about real software systems even though methods for doing so have greatly improved since 1996. On the other hand, lightweight formal methods are widely used; static analysis can rapidly detect a wide variety of shallow errors that would otherwise result in difficult debugging sessions.
