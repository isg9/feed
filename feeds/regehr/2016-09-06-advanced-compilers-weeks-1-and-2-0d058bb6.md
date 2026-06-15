---
title: Advanced Compilers Weeks 1 and 2
url: https://blog.regehr.org/archives/1419
published: "2016-09-06T04:22:25Z"
feed: regehr
guid: http://blog.regehr.org/?p=1419
---

# Advanced Compilers Weeks 1 and 2

This post will be of somewhat narrow interest; it’s a quick attempt to take my lecture notes for the first weeks of an advanced compilers course and turn them into something a bit more readable. I’m not using slides for this class.

# Motivation

The great thing about an advanced course (on any topic) is that we have a lot of freedom in choosing the direction that the class takes. My class this fall is mainly about static program analysis: predicting the behavior of programs without running them. This is a broadly useful technology, it is the foundation for type checking, program verification, compiler optimization, and static bugfinding.

We can start off with a couple of observations about the role of compilers. First, hardware is getting weirder rather than getting clocked faster: almost all processors are multicores and it looks like there is increasing asymmetry in resources across cores. Processors come with vector units, crypto accelerators, bit twiddling instructions, and lots of features to make virtualization and concurrency work. We have DSPs, GPUs, big.little, and Xeon Phi. This is only scratching the surface. Second, we’re getting tired of low-level languages and their associated security disasters, we want to write new code, to whatever extent possible, in safer, higher-level languages. Compilers are caught right in the middle of these opposing trends: one of their main jobs is to help bridge the large and growing gap between increasingly high-level languages and increasingly wacky platforms. It’s effectively a perpetual employment act for solid compiler hackers.

The [sufficiently smart compiler](http://c2.com/cgi/wiki?SufficientlySmartCompiler) never seems to arrive. I told the class [a story that I never tire of re-telling](https://blog.regehr.org/archives/1049). My understanding is that while the death of the Cell processor was complicated (yields were bad, GPUs were on the rise, etc.) the lack of good tooling certainly didn’t help. Perhaps later on we’ll read [this paper](http://groups.csail.mit.edu/cag/crg/papers/eichenberger05cell.pdf).

# Semantics

One of the big ideas that enables static program analysis is that programs mean something, mathematically speaking. Of course this was understood very early by the people who created computer science, but in the early history of compilers people would get tripped up by the fact that they didn’t necessarily have a good idea what the programs being compiled actually meant. A new optimization would break programs and it wasn’t possible to assign blame cleanly: was the program within its rights to expect a certain behavior or not? This kind of question can only be answered by assigning meaning to programs. Alas, it is still common for a program to mean “whatever the (single) language implementation does with the program.” I’ve heard stories from Matlab users that the providers of the Matlab implementation have introduced subtle changes to the semantics over time, probably both intentionally and unintentionally. The alternative to defining the semantics using an implementation is to define the semantics of a language some other way, either in a standards document or in math. Then, both programs and implementations can be judged to be either in conformance, or not, with the standard. Obviously this is no panacea, as long experience with C and C++ has shown — but it’s better than nothing.

There are a lot of ways to write down the semantics of a programming language but an even more important issue is creating an appropriate semantics. For example, a language designed for implementing constant-time cryptography might include execution time in the semantics. A language for embedded systems might include memory allocation (or at least guarantees about the lack of implicit allocations) in the semantics. Even the simple parts of a language, such as arithmetic, contain many subtle corners. [Here’s an example](https://blog.regehr.org/archives/482). We can also look at the behavior of shift operators when the shift exponent is at least as large as the width of the shifted value. Java and x86 reduce the shift amount modulo 32. ARM reduces the shift amount modulo 256 and then saturates (shift by 257 is equivalent to shift by 1 but shift by 100 is equivalent to clearing the register). C and C++ have (of course!) undefined semantics for shift by 100 or 257. Constraining the semantics is nice but too many constraints make efficient code generation difficult. The WebAsm people were [discussing these issues](https://github.com/WebAssembly/design/issues/350) not too long ago. I’ve always wanted shift left by -3 to be a shift right by 3, but nobody else has ever thought this was a good idea, as far as I know.

The recent [DAO debacle](http://www.coindesk.com/understanding-dao-hack-journalists/) provided an absolutely wonderful demonstration of why it might be risky to define the semantics of a language using a reference implementation. They put a lot of money on the line there, the hubris was impressive. One hopes that lessons were learned.

The overall point of this discussion is (1) we can’t do static program analysis unless we know what the programming language means and (2) designing meanings for programs is an interesting and difficult topic in itself.

# Missed Optimizations

I asked the students to use the [Compiler Explorer](https://gcc.godbolt.org/) to demonstrate a case in which each of GCC and LLVM miss an optimization, and to provide the assembly code that the compiler should have generated. We went over a handful of submissions, discussing the issues: Was the proposed optimization correct? Would it be a good idea to implement it now? What kind of static analysis would be needed to make the optimization go?

As I had hoped, the codes written by the students exposed many interesting issues. One example that came up was similar to [this one](https://godbolt.org/g/pYJlij) where LLVM cleverly realizes that the loop is squaring the function but then (apparently) fails to remove the subsequent conditional move. But really, since the loop fails to execute when the argument is negative, some sort of conditional really is needed. We also saw some good examples where potential aliasing was blocking optimizations. Playing with optimizations in compiler explorer is really a pleasure.

# Intro to Static Analysis

Although there are a lot of slide decks that do a good job explaining static analysis, there’s [only one book-length treatment of the subject that I like](https://cs.au.dk/~amoeller/spa/spa.pdf), which I’ll call SPA. SPA is clearly written, it avoids unnecessary notation, and it keeps the material grounded in practical use cases. It’s great!

I started out using everyone’s favorite tutorial abstract domains: parity (are values even or odd?) and signs (are values negative, zero, or positive?). I introduced what I consider to be the first key idea behind static analysis, which is that abstract values (odd, positive, etc.) are simply stand-ins for sets of concrete values. This is such a simple idea and yet it can get lost if the material is presented wrong. We discussed some transfer functions such as addition for the even/odd domain and multiplication for the signedness domain (as seen on p. 28 of SPA). Here the key idea is that we can always verify a transfer function by concretizing the abstract arguments, applying the concrete operation pairwise to the sets of concrete values (assuming a binary operator), and then lifting the result set back into the abstract domain. This now sets the stage for introducing the abstraction and concretization functions and then we’re ready for the Galois connection (which I showed the components of but did not explicitly name). [David Schmidt’s slides on this material](http://santos.cs.ksu.edu/schmidt/Escuela03/WSSA/talk1p.pdf) are awesome.

The thing that we’re working up to here is digging into some of the numerous static analyses that are part of LLVM. I’m trying to introduce the theory, which is very beautiful, while also warming the students up to the idea that it all sort of goes out the window when you’re confronted with the piles of C++ that actually make these analyses happen in practice.

# Types

Everyone read Chapter 3 of SPA as well as the first section of [Type Systems](http://lucacardelli.name/papers/typesystems.pdf), another piece of writing that I like very much because it keeps the topics connected to the reasons why they are useful. I didn’t want to get into type systems too deeply (and in fact types are something of a non-speciality of mine) but did want to students to come away with the idea that type checking is an important use case of static program analysis.

The point of static typechecking is that “well typed programs can’t go wrong” but as Cardelli points out in some detail, we need to be pretty careful when saying what “go wrong” means. He includes some nice discussion of the standard static/dynamic and safe/unsafe language categorizations.
