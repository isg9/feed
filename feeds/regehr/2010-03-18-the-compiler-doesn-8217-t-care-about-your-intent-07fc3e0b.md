---
title: The Compiler Doesn&#8217;t Care About Your Intent
url: https://blog.regehr.org/archives/47
published: "2010-03-18T18:49:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=47
---

# The Compiler Doesn&#8217;t Care About Your Intent

A misunderstanding that I sometimes run into when teaching programming is that the compiler can and should guess what the programmer means.  This isn’t usually quite what people say, but it’s what they’re thinking.  A great example appeared in a message sent to the avr-gcc mailing list.  The poster had upgraded his version of GCC, causing a delay loop that previously worked to be completely optimized away.  (Here’s the [original message](http://www.mail-archive.com/avr-gcc-list@nongnu.org/msg04031.html) in the thread and [the message I’m quoting from](http://www.mail-archive.com/avr-gcc-list@nongnu.org/msg04068.html).)  The poster said:

> … the delay loop is just an example, of how simple, intuitive code can throw the compiler into a tizzy. I’ve used SDCC(for mcs51) where the compiler ‘recognises’ code patterns, and says “Oh, I know what this is – it’s a delay loop! – Let it pass.”(for example).

Another example comes from the pre-ANSI history of C, before the volatile qualifier existed.  The [compiler would attempt to guess whether the programmer was accessing a hardware register](http://groups.google.com/group/comp.std.c/msg/7709e4162620f2cd), in order to avoid optimizing away the critical accesses.  I’m sure there are more examples out there (if you know of a good one, please post a comment or mail me).

The problem with this kind of thinking is that the correctness of code now depends on the compiler correctly guessing what the programmer means.  Since the heuristics used to guess aren’t specified or documented, they are free to change across compilers and compiler versions.  In the short term, these hacks are attractive, but in the long run they’re little disasters waiting to happen.

One of the great innovations of the past 50 years of programming language research was to separate the semantics of a language from its implementation.  This permits the correctness of an implementation to be judged solely by how closely it conforms to the standard, and it also permits programs to be reasoned about as mathematical objects.  C is not the most suited to mathematical reasoning, but there are some excellent research projects that do exactly this.  For example [Michael Norrish’s PhD thesis](http://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-453.html) formalized C in the HOL theorem prover, and Xavier Leroy’s [CompCert compiler](http://compcert.inria.fr/) provably preserves the meaning of a C program as it is translated into PPC or ARM assembly.

Of course, the intent of the programmer does matter sometimes.  First, a well-designed programming language takes programmer intent into account and makes programs mean what they look like they mean.  Second, intent is important for readability and maintainability.  In other words, there are usually many ways to accomplish a given task, and good programmers choose one that permits subsequent readers of the code to easily grasp what is being done, and why.  But the compiler does not, and should not, care about intent.
