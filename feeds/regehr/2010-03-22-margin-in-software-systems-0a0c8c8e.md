---
title: Margin in Software Systems
url: https://blog.regehr.org/archives/50
published: "2010-03-22T23:56:26Z"
feed: regehr
guid: http://blog.regehr.org/?p=50
---

# Margin in Software Systems

*Margin of safety* is a fundamental engineering concept where a system is built to tolerate loads exceeding the maximum expected load by some factor.  For example, structural elements of buildings typically have a margin of safety of 100%: they can withstand twice the expected maximum load.  Pressure vessels have more margin, in the range 250%-300%, whereas the margin for airplane landing gear may be only 25%.  (All these examples are from the [Wikipedia article](http://en.wikipedia.org/wiki/Factor_of_safety).)

We can say that a software system has a margin of safety *S* with respect to some external load or threat *L* only when the expected maximum load *Lmax* can be quantified and the system can be shown to function properly when subjected to a load of (1+ *S*) *Lmax*.  Software systems are notoriously low on margin: a single flaw will often compromise the entire system.  For example, a buffer overflow vulnerability in a networked application can permit an attacker to run arbitrary code at the same privilege level as the application, subverting its function and providing a vector of attack to the rest of the system.

Software is often defined to be correct when, for every input, it produces the correct output.  Margin is an orthogonal concern.  For example, there exist systems that are formally verified to be correct, such as [CompCert](http://compcert.inria.fr/) and [seL4](http://ertos.nicta.com.au/research/sel4/), that have no margin at all with respect to flaws not covered by the proof — a bug or trojan in the assembler, linker, or loader invalidates the safety argument of either system.  Similarly, there exist systems that are obviously not correct, that have considerable margin.  For example, my Windows laptop has enough RAM that it can tolerate memory leak bugs for quite a while before finally becoming unusable.

There are other examples of margin in software systems:

- Many classes of real-time systems have a characteristic *utilization bound*: a fraction of total available CPU time that, if not exceeded, permits all sub-computations to meet their time constraints.  Real safety-critical systems are usually designed to use less CPU time than their theoretical utilization bounds, providing margin against spurious interrupts or other unforeseen demands on the processor.
- Forward error correction provides margin against data corruption.
- n-version programming and replication provide margin respectively against software and hardware defects.
- It is common to pick a cryptographic key larger than the smallest currently-unbreakable key, providing some margin against future improvements in processing power and cryptanalysis.

The piece that is missing, as far as I can tell, is a collection of broader results about how margin in software systems relates to overall system reliability, and how useful kinds of software margin can be obtained at acceptable cost.

What are some general ways to gain margin?  Overprovisioning a resource, as in the examples above, is very common.  Defense in depth is also important: many software systems have only two lines of defense against attack: safety checks at the application level, and safety checks in the OS kernel.  If both of these defenses fall — as is common — the entire system has been subverted.  Virtual machines, safe language runtimes, and similar mechanisms can add layers of defense, as can firewalls and other external barriers.

Something I want to see is “margin-aware formal methods.”  That is, ways to reason about software systems under attack or load.  The result, ideally, would be analogous to well-understood principles of [safety engineering](http://en.wikipedia.org/wiki/Safety_engineering).  Examples already exist:

- [Symbolic robustness analysis](http://www.cs.ucla.edu/~rupak/Papers/Symbolic_robustness_analysis.ps) is an early proof-of-concept technique for showing that small perturbations in the input to a control system result in only small changes to the control signal
- The [critical scaling factor](http://www.cc.gatech.edu/classes/AY2008/cs4220_fall/5A-RM-Average.pdf) in scheduling theory is the largest degree of slowdown computations can incur before deadlines start being missed
- [Byzantine fault tolerance](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance) is a body of theory showing that a distributed system can produce correct results if fewer than a third of its nodes are compromised

An important obstacle to margin in software systems is the high degree of coupling between components.  Coupling can be obvious, as when multiple activities run in the same address space, or it can be insidiously indirect, including common failure modes in independent implementations, as [Knight and Leveson observed in 1986](http://portal.acm.org/citation.cfm?id=10688).  There are many other kinds of coupling, including reliance on multiple copies of a flawed library, a single password used across multiple systems, etc.  It can be very hard to rule out all possible forms of coupling between components — for example, even if we use Byzantine fault tolerance and n-version programming, we can still be compromised if all chips come from the same (faulty or malicious) fab.

In summary: engineered physical systems almost always have margin with respect to failures.  Too often, software does not.  This should be fixed.  I want my OS and web browser to each come with a statement such as “We estimate that no more than 25 exploitable buffer overflows remain in this product, therefore we have designed it to be secure in the presence of up to 70 such problems.”
