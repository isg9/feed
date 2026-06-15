---
title: 'Poll: Do You Want Compiler Bugs to Be Quiet or Loud?'
url: https://blog.regehr.org/archives/238
published: "2010-08-09T19:17:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=238
---

# Poll: Do You Want Compiler Bugs to Be Quiet or Loud?

I’m preparing a longer piece on compiler correctness but thought this idea was worth putting into its own short post. The question is:

> Which would you choose: A compiler that crashes more often, or one that silently generates incorrect code more often?

Of course, all real compilers have bugs, and all real compilers display both kinds of symptoms. Typically these bugs start to show up when a project pushes the boundaries a bit. People who develop embedded software often encounter both kinds of bugs depressingly often.

I suspect that many readers will have a gut preference for the crashy compiler, based on the idea that it’s better to know early when something is wrong with the compiler. But consider that if you need to ship a product, the crashy compiler is likely to be a showstopper until a workaround is found. Upgrading the compiler is usually totally unacceptable, so the workaround has to be on the application side. On the other hand, a bit of wrong code in some module is probably not a showstopper: development and testing of the rest of the system can proceed.

It is sometimes possible to choose between failure modes for a given bug. [Failure oblivious computing](http://people.csail.mit.edu/rinard/paper/osdi04.pdf) (FOC) is based on the idea that we prefer a highly available system over a crashy one. Applied to a compiler, FOC would (at least sometimes) change crash errors into wrong code errors. I’ve talked to compiler developers who use a very FOC-like approach to avoid compiler crashes; they had a compiler pass that would walk various data structures repairing anything that looked fishy. These were smart people and they decided to spend their time masking bugs rather than fixing them. In contrast, other compiler developers use assertions to convert wrong code bugs into crash bugs, and to change dirty crash bugs (segfaults) into clean crashes. What I’m really asking here is not what kind of bugs people should put into code — we have little enough control over that! — but whether we want developers to invest in systems for making bugs show up earlier (assertions) or not at all (FOC).

\[poll id=”2″\]
