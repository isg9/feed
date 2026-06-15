---
title: 'Memory Safe C/C++: Time to Flip the Switch'
url: https://blog.regehr.org/archives/939
published: "2013-04-23T18:25:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=939
---

# Memory Safe C/C++: Time to Flip the Switch

For a number of years I’ve been asking:

> If the cost of memory safety bugs in C/C++ codes is significant, and if solutions are available, why aren’t we using them in production systems?

Here’s a [previous blog post](https://blog.regehr.org/archives/715) on the subject and a quick summary of the possible answers to my question:

- The cost of enforcement-related slowdowns is greater than the cost of vulnerabilities.
- The cost due to slowdown is not greater than the cost of vulnerabilities, but people act like it is because the performance costs are up-front whereas security costs are down the road.
- Memory safety tools are not ready for prime time for other reasons, like maybe they crash a lot or raise false alarms.
- Plain old inertia: unsafety was good enough 40 years ago and it’s good enough now.

I’m returning to this topic for two reasons. First, there’s a new paper [SoK: Eternal War in Memory](http://www.cs.berkeley.edu/~dawnsong/papers/Oakland13-SoK-CR.pdf) that provides a useful survey and analysis of current methods for avoiding memory safety bugs in legacy C/C++ code. (I’m probably being dense but can someone explain what “SoK” in the title refers to? In any case I like the [core war](http://en.wikipedia.org/wiki/Core_War) allusion.)

When I say “memory safety” I’m referring to relatively comprehensive strategies for trapping the subset of undefined behaviors in C/C++ that are violations of the memory model and that frequently lead to RAM corruption (I say “relatively comprehensive” since even the strongest enforcement has holes, for example due to inline assembly or libraries that can’t be recompiled). The paper, on the other hand, is about a broader collection of solutions to memory safety problems including weak ones like ASLR, stack canaries, and NX bits that catch small but useful subsets of memory safety errors with very low overhead.

The SoK paper does two things. First, it analyzes the different pathways that begin with an untrapped undefined behavior and end with an exploit. This analysis is useful because it helps us understand the situations in which each kind of protection is helpful. Second, the paper evaluates a collection of modern protection schemes along the following axes:

- protection: what policy is enforced, and how effective is it at stopping memory-based attacks?
- cost: what is the resource cost in terms of slowdown and memory usage?
- compatibility: does the source code need to be changed? does it need to be recompiled? can protected and unprotected code interact freely?

As we might expect, stronger protection generally entails higher overhead and more severe compatibility problems.

The second reason for this post is that I’ve reached the conclusion that 30 years of research on memory safe C/C++ should be enough. It’s time to suck it up, take the best available memory safety solution, and just turn it on by default for a major open-source OS distribution such as Ubuntu. For those of us whose single-user machines are quad-core with 16 GB of RAM, the added resource usage is not going to make a difference. I promise to be an early adopter. People running servers might want to turn off safety for the more performance-critical parts of their workloads (though of course these might be where safety is most important). Netbook and Raspberry Pi users probably need to opt out of safety for now.

If the safe-by-default experiment succeeded, we would have (for the first time) a substantial user base for memory-safe C/C++. There would then be an excellent secondary payoff in research aimed at reducing the cost of safety, increasing the strength of the safety guarantees, and dealing with safety exceptions in interesting ways. My guess is that progress would be rapid. If the experiment failed, the new OS would fail to gain users and the vendor would have to back off to the unsafe baseline.

Please nobody leave a comment suggesting that it would be better to just stop using C/C++ instead of making them safe.
