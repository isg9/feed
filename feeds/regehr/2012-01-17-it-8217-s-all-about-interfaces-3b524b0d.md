---
title: It&#8217;s All About Interfaces
url: https://blog.regehr.org/archives/666
published: "2012-01-17T05:09:51Z"
feed: regehr
guid: http://blog.regehr.org/?p=666
---

# It&#8217;s All About Interfaces

The [Frank system](http://www.vpri.org/pdf/tr2011004_steps11.pdf) — see also [this recent post](https://blog.regehr.org/archives/663) — is intended to reduce the amount of code needed to create a usable desktop software stack by about 1000x. I’m pretty sure that this goal is sufficient but not necessary. In other words, if we can reduce software size to 0.1% of its current size then that’s great and a lot of current software problems will be far less severe. However, the 1000x reduction is probably unnecessarily difficult — there may be easier ways to achieve similar benefits.

To support my claim that “1000x reduction in software size” is too strong of a goal, consider this question: Would a reasonable person object to running Frank on a modern Intel processor containing more than two billion transistors? In other words, does the gigantic complexity of one of these chips somehow detract from Frank’s simplicity? Of course not. The x86-64 ISA forms a fairly leak-proof abstraction boundary between Frank and all those transistors.\[1. Is the ISA exported by a modern Intel chip really leak-proof? Pretty much so as far as the functional semantics goes. However, the abstraction is incredibly leaky with respect to performance — but this only matters for real-time systems and performance-sensitive codes — neither of which is in Frank’s target domain.\] I believe the same argument can be made about other interfaces. Let’s say that we have a compiler or an operating system that we trust, perhaps because it has been proved correct. In this case, does it really matter how much code implements the proved-correct interface? I don’t think so.

What we need, then, are more interfaces that are:

- durable
- non-leaky
- beautiful (or, at least, extremely workable)

A durable interface lasts. For example, the UNIX system call interface is reasonably durable — it is often the case that a statically linked UNIX executable will run on many years’ worth of systems, whereas a dynamically linked program rots fairly quickly. x86 has proved to be highly durable.

A non-leaky interface reveals little or nothing about the interface’s implementation. Trivially, a non-leaky interface requires some sort of language-based or OS-based memory isolation — otherwise peeks and pokes (and their unintentional equivalents) cause leaks. Non-leaky interfaces require a lot of error-checking code to ensure that each side adheres to its part of the contract. Performance leaks, alas, are nearly impossible to avoid, unless we give up on implementing optimizations. Bugs are perhaps the ultimate abstraction leak, forcing evil workarounds in code using the interface. Formal verification and automatically-inserted contract checks constitute partial solutions.

Beautiful interfaces do not expose too much or too little functionality, nor are they more complex than they need to be. I’m not sure that I’ve seen a lot of specific examples of beautiful interfaces lately, but at a slightly more abstract level I would consider regular expressions, database transactions, stream sockets, [SMT queries](http://www.smtlib.org/), RISC instruction sets, functional programming languages, and hierarchical filesystems to all have a certain amount of beauty. A lot of work and a lot of iteration is required to create a beautiful interface — so this conflicts somewhat with durability.

The worthwhile research problem, then, is to create interfaces having these properties in order that the code living in between interface layers can be developed, understood, debugged, and maintained in a truly modular fashion. To some extent this is a pipe dream since the “non-leaky” requirement requires both correctness and performance opacity: both extremely hard problems. Another problem with this idea — from the research point of view — is that a grant proposal “to create durable, non-leaky, beautiful interfaces” is completely untenable, nor is it even clear that most of this kind of work belongs in academia. On the other hand, it seems clear that we don’t want to just admit defeat either. If we disregard the people who are purely chasing performance and those who are purely chasing correctness, a substantial sub-theme in computer systems research can be found where people are chasing beautiful interfaces. There are probably a lot of good examples I could give here, but Matthew Flatt ( [example](http://www.cs.utah.edu/plt/publications/pldi04-ff.pdf)) and Eddie Kohler ( [example](http://www.read.seas.harvard.edu/~kohler/pubs/mammarella09modular.pdf)) come to mind.

In summary, I’ve tried to argue that creating a really small software stack, as in Frank, is a good goal, but not necessarily the very best one.
