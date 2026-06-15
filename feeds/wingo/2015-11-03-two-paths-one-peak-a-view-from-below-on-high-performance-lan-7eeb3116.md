---
title: 'two paths, one peak: a view from below on high-performance language implementations — wingolog'
url: https://wingolog.org/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations
published: "2015-11-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations
---

# two paths, one peak: a view from below on high-performance language implementations — wingolog

## [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)

3 November 2015 11:47 PM

- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [optimization](/tags/optimization)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [scheme](/tags/scheme)
- [adaptive optimization](/tags/adaptive%20optimization)
- [inlining](/tags/inlining)
- [javascript](/tags/javascript)

Ohmigod it's November. Time flies amirite. Eck-setra. These are not actually my sentiments but sometimes I do feel like a sloth or a slow loris, grasping out at quarter-speed. Once I get a hold it's good times, but hoo boy. The tech world churns and throws up new languages and language implementations every year and how is it that in 2015, some 20 years after the project was started, Guile still doesn't do native compilation?

Though I've only been Guiling for the last 10 years or so, this article aims to plumb those depths; and more than being an apology or a splain I want to ponder the onward journey from the here and the now. I was going to write something like "looking out from this peak to the next higher peak" but besides being a cliché that's exactly what I don't mean to do. In Guile performance work I justify my slow loris grip by a mistrust of local maxima. I respect and appreciate the strategy of going for whatever gains you the most in the short term, especially in a commercial context, but with a long view maybe this approach is a near win but a long lose.

That's getting ahead of myself; let's get into this thing. We started byte-compiling Guile around 2008 or so. Guile is now near to native compilation. Where are we going with this thing?

**short term: template jit**

The obvious next thing to do for Guile would be to compile its bytecodes to machine code using a template JIT. This strategy just generates machine code for each bytecode instruction without regard to what comes before or after. It's dead simple. Guile's bytecode is quite well-suited to this technique, even, in the sense that an instruction doesn't correspond to much code. As Guile has a register-based VM, its instructions will also specialize well against their operands when compiled to native code. The only global state that needs to be carried around at runtime is the instruction pointer and the stack pointer, both of which you have already because of how modern processors work.

Incidentally I have long wondered why CPython doesn't have a template JIT. Spiritually I am much more in line with the PyPy project but if I were a CPython maintainer, I would use a template JIT on the bytecodes I already have. Using a template JIT preserves the semantics of bytecode, including debugging and introspection. CPython's bytecodes are at a higher level than Guile's though, with many implicit method/property lookups (at least the last time I looked at them), and so probably you would need to add inline caches as well; but no biggie. Why aren't the CPython people doing this? What is their long-term perf story anyway -- keep shovelling C into the extension furnace? Lose to PyPy?

In the case of Guile we are not yet grasping in this direction because we don't have (direct) competition from PyPy :) But also there are some problems with a template JIT. Once you internalize the short-term mentality of a template JIT you can get stuck optimizing bytecode, optimizing template JIT compilation, and building up a baroque structure that by its sheer mass may prevent you from ever building The Right Thing. You will have to consider how a bytecode-less compilation pipeline interacts with not only JITted code but also bytecode, because it's a lose to do a template JIT for code that is only executed once.

This sort of short-term thinking is what makes people also have to support on-stack replacement (OSR), also known as hot loop transfer. The basic idea is that code that executes often has to be JITted to go fast, but you can't JIT everything because that would be slow. So you wait to compile a function until it's been called a few times; fine. But with loops it could be that a function is called just once but a loop in the function executes many times. You need to be able to "tier up" to the template JIT from within a loop. This complexity is needed at the highest performance level, but if you choose to do a template JIT you're basically committing to implementing OSR early on.

Additionally the implementation of a template JIT compiler is usually a bunch of C or C++ code. It doesn't make sense to include a template JIT in a self-hosted system that compiles to bytecode, because it would be sad to have the JIT not be written in the source language (Guile Scheme in our case).

Finally in Scheme we have tail-call and delimited continuation considerations. Currently in Guile all calls happen in the Guile bytecode interpreter, which makes tail calls easy -- the machine frame stays the same and we just have to make a tail call on the Scheme frame. This is fine because we don't actually control the machine frame (the C frame) of the bytecode interpreter itself -- the C compiler just does whatever it does. But how to tail call between the bytecode interpreter and JIT-compiled code? You'd need to add a trampoline beneath both the C interpreter and any entry into compiled code that would trampoline to the other implementation, depending on how the callee "returns". And how would you capture stack slices with delimited continuations? It's possible (probably -- I don't know how to reinstate a delimited continuation with both native and interpreted frames), but something of a headache, and is it really necessary?

**if you compile ahead-of-time anyway...**

The funny thing about CPython is that, like Guile, it is actually an ahead-of-time compiler. While the short-term win would certainly be to add a template JIT, because the bytecode is produced the first time a script is run and cached thereafter, you might as well compile the bytecode to machine code ahead-of-time too and skip the time overhead of JIT compilation on every run. In a template JIT, the machine code is only a function of the bytecode (assuming the template JIT doesn't generate code that depends on the shape of the heap).

Compiling native code ahead of time also saves on memory usage, because you can use [file-backed mappings that can be lazily paged in and shared between multiple processes](https://wingolog.org/archives/2014/01/19/elf-in-guile), and when these pages are in cache that translates also to faster startup too.

But if you're compiling bytecode ahead of time to native code, what is the bytecode for?

**(not) my beautiful house**

At some point you reach a state where you have made logical short-term decisions all the way and you end up with vestigial organs of WTF in your language runtime. Bytecode, for example. A bytecode interpreter written in C. Object file formats for things you don't care about. Trampolines. It's time to back up and consider just what it is that we should be building.

The highest-performing language implementations will be able to compile together the regions of code in which a program spends most of its time. Ahead-of-time compilers can try to predict these regions, but you can't always know what the behavior of a program will be. A program's run-time depends on its inputs, and program inputs are late-bound.

Therefore these highest-performing systems will use some form of adaptive optimization to apply run-time JIT compilation power on whatever part of a program turns out to be hot. This is *the* peak performance architecture, and indeed in the climb to a performant language implementation, there is but one peak that I know of. The question becomes, how to get there? What path should I take, with the priorities I have and the resources available to me, which lets me climb the farthest up the hill while always leaving the way clear to the top?

**guile's priorities**

There are lots of options here, and instead of discussing the space as a whole I'll just frame the topic with some bullets. Here's what I want out of Guile:

1. The project as a whole should be pleasing to hack on. As much of the system as possible should be written in Scheme, as little as possible in C or assembler, and dependencies on outside projects should be minimized.

2. Guile users should be able to brag about startup speed to their colleagues. We are willing to trade away some peak throughput for faster startup, if need be.

3. Debuggability is important -- a Guile hacker will always want to be able to get stack traces with actual arguments and local variable values, unless they stripped their compiled Guile binaries, which should be possible as well. But we are willing to give up some debuggability to improve performance and memory use. In the same way that a tail call replaces the current frame in its entirety, we're willing to lose values of dead variables in stack frames that are waiting on functions to return. We're also OK with other debuggability imprecisions if the performance gains are good enough. With macro expansion, Scheme hackers expect a compilation phase; spending time transforming a program via ahead-of-time compilation is acceptable.

Call it the Guile Implementor's Manifesto, or the manifesto of this implementor at least.

**beaucoup bucks**

Of course if you have megabucks and ace hackers, then you want to dial back on the compromises: excellent startup time but also source-level debugging! The user should be able to break on any source position: the compiler won't even fold `1 + 1` to `2`. But to get decent performance you need to be able to tier up to an optimizing compiler soon, and soon in two senses: soon after starting the program, but also soon after starting your project. It's an intimidating thing to build when you are just starting on a language implementation. You need to be able to tier down too, at least for debugging and probably for other reasons too. This strategy goes in the right direction, performance-wise, but it's a steep ascent. You need experienced language implementors, and they are not cheap.

The usual strategy for this kind of implementation is to write it all in C++. The latency requirements are too strict to do otherwise. Once you start down this road, you never stop: your life as an implementor is that of a powerful, bitter C++ wizard.

The PyPy people have valiently resisted this trend, writing their Python implementation in Python itself, but they concede to latency by compiling their "translated interpreter" into C, which then obviously can't itself be debugged as Python code. It's self-hosting, but staged into C. Ah well. Still, a most valiant, respectable effort.

This kind of language implementation usually has bytecode, as it's a convenient reification of the source semantics, but it doesn't have to. V8 is a good counterexample, at least currently: it treats JavaScript source code as the canonical representation of program semantics, relying on its ability to re-parse source text to an AST in the same way every time as needed. V8's first-tier implementation is actually a simple native code compiler, generated from an AST walk. But things are moving in the bytecode direction in the V8 world, reluctantly, so we should consider bytecode as the backbone of the beaucoup-bucks language implementation.

**shoestring slim**

If you are willing to relax on source-level debugging, as I am in Guile, you can simplify things substantially. You don't need bytecode, and you don't need a template JIT; in the case of Guile, probably the next step in Guile's implementation is to replace the bytecode compiler and interpreter with a simple native code compiler. We can start with the equivalent of a template JIT, but without the bytecode, and without having to think about the relationship between compiled and (bytecode-)interpreted code. (Guile still has a traditional tree-oriented interpreter, but it is actually written in Scheme; that is a story for another day.)

There's no need to stop at a simple compiler, of course. Guile's bytecode compiler is already fairly advanced, with interprocedural optimizations like [closure optimization](http://lambda-the-ultimate.org/node/5259), [partial evaluation](https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile), and contification, as well as the usual [loop-invariant code motion](https://wingolog.org/archives/2015/07/28/loop-optimizations-in-guile), [common subexpression elimination](https://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile), scalar replacement, unboxing, and so on. Add register allocation and you can have quite a fine native compiler, and you might even beat the fabled Scheme compilers on the odd benchmark. They'll call you plucky: high praise.

There's a danger in this strategy though, and it's endemic in the Scheme world. Our compilers are often able to do heroic things, but only on the kinds of programs they can fully understand. We as Schemers bend ourselves to the will of our compilers, writing only the kinds of programs our compilers handle well. Sometimes we're scared to `fold`, preferring instead to inline the named-let iteration manually to make sure the compiler can do its job. We `fx+` when we should `+`; we use tagged vectors when we should use proper data structures. This is *déformation professionelle*, as the French would say. I gave a [talk at last year's Scheme workshop](https://wingolog.org/archives/2014/11/27/scheme-workshop-2014) on this topic. PyPy people largely don't have this problem, for example; their langauge implementation is able to see through abstractions at run-time to produce good code, but using adaptive optimization instead of ahead-of-time trickery.

So, an ahead-of-time compiler is perhaps a ridge, but it is not the peak. No amount of clever compilation will remove the need for an adaptive optimizer, and indeed too much cleverness will stunt the code of your users. The task becomes, how to progress from a decent AOT native compiler to a system with adaptive optimization?

Here, as far as I know, we have a research problem. In Guile we have mostly traced the paths of history, re-creating things that existed before. As Goethe said, quoted in the introduction to The Joy of Cooking: "That which thy forbears have bequeathed to thee, earn it anew if thou wouldst possess it." But finally we find here something new, or new-ish: I don't know of good examples of AOT compilers that later added adaptive optimization. Do you know of any, dear reader? I would be delighted to know.

In the absence of a blazed trail to the top, what I would like to do is to re-use the AOT compiler to do dynamic inlining. We might need to collect type feedback as well, though inlining is the more important optimization. I think we can serialize the compiler's intermediate representation into a special section in the ELF object files that Guile produces. A background thread or threads can monitor profiling information from main threads. If a JIT thread decides two functions should be inlined, it can deserialize compiler IR and run the standard AOT compiler. We'd need a bit of mutability in the main program in which to inject such an optimization; an inline cache would do. If we need type feedback, we can make inline caches do that job too.

All this is yet a ways off. The next step for Guile, after the 2.2 release, is a simple native compiler, then register allocation. Step by step.

**but what about llvmmmmmmmmmmmmm**

People always ask about LLVM. It is an *excellent* compiler backend. It's a bit big, and maybe you're OK with that, or maybe not; whatever. Using LLVM effectively depends on your ability to deal with churn and big projects. But if you can do that, swell, you have excellent code generation. But how does it help you get to the top? Here things are less clear. There are a few projects using LLVM effectively as a JIT compiler, but that is a very recent development. My hubris, desire for self-hosting, and lack of bandwidth for code churn makes it so that I won't use LLVM myself but I have no doubt that a similar strategy to that which I outline above could work well for LLVM. Serialize the bitcode into your object files, make it so that you can map all optimization points to labels in that bitcode, and you have the ability to do some basic dynamic inlining. Godspeed!

**references**

If you're interested, I gave a talk a year ago on [the state of JavaScript implementations](https://wingolog.org/archives/2014/12/09/state-of-js-implementations-2014-edition), and how they all ended up looking more or less the same. This common architecture was first introduced by [Self](https://www.cs.ucsb.edu/~urs/oocsb/self/papers/dynamic-deoptimization.html); languages implementations in this category include HotSpot and any of the JavaScript implementations.

[Some notes on how PyPy produces interpreters from RPython.](https://rpython.readthedocs.org/en/latest/translation.html)

**and so I bid you good night**

Guile's compiler has grown slowly, in tow of my ballooning awareness of ignorance and more slowly inflating experience. Perhaps we could have done the native code compilation thing earlier, but I am happy with our steady progress over the last five years or so. We had to scrap one bytecode VM and one or two compiler intermediate representations, and though that was painful I think we've done pretty well as far as higher-order optimizations go. If we had done native compilation earlier, I can't but think the inevitably wrong decisions we would have made on the back-end would have prevented us from having the courage to get the middle-end right. As it is, I see the way to the top, through the pass of ahead-of-time compilation and thence to a dynamic inliner. It will be some time before we get there, but that's what I signed up for :) Onward!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [scheme workshop 2014](/archives/2014/11/27/scheme-workshop-2014)
- [partial evaluation in guile](/archives/2011/10/11/partial-evaluation-in-guile)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)

### 12 responses

1. [Ian McKellar](http://ian.mckellar.org/) says:[4 November 2015 1:34 AM](#45702ce79ea97a059e574ee2688d3e34dc0f2df4)

   There was an effort (mostly out of Google) to add a JIT to CPython using LLVM. I even hacked on it a bit - I think I was working on a JIT for Python regular expressions when we were galavanting around Barcelona six years ago. My regex JIT never worked well and the project as a whole was abandoned at some point.

   http://www.infoq.com/news/2011/03/unladen-swallow

2. [John Cowan](http://www.ccil.org/~cowan) says:[4 November 2015 2:44 AM](#8a093e508147b6c44826c11f76f67fd9b2a736be)

   *why CPython doesn't have a template JIT*

   Because it wouldn't be dead simple. CPython is all about simple, stupid implementations. Python lists are like Lisp lists in how they are used, but they are implemented as C arrays, and when you need to insert or delete or even get the cdr of the list, you copy all the rest of it. Thassit. Everything that can be done that way, *is* done that way.

   *What is their long-term perf story anyway?*

   Three words will cover it. "We Don't Care". Python is not a Fortran or a Lisp or a Smalltalk, that was mocked early on for being "too slow to be usable". [Python is a Cobol](https://en.wikipedia.org/wiki/Nately#Background_information), which was able to escape the snipers until various degrees of Moore's Law made it safe for democracy. (Follow the link if you don't get the Catch-22 reference.)

3. Alex Elsayed says:[4 November 2015 7:32 AM](#5fd847038f2f6008a3a4c07e4eb2cd013f03c40b)

   One thing you might find interesting is Thorin\[1\], which grew out of the AnyDSL project\[2\]. It's a higher-order intermediate language, with first-class continuations combined with a graph representation of values.

   While the existing codebase for it is C++, and it currently backends on LLVM, the codebase is not large and thus could likely be reimplemented without much ado, and it looks very amenable to direct generation - possibly even staging from interpreting the IR -> bytecode -> JIT, as its CPS nature means that even for loops one can stage optimization levels on function entry.

   And even if a direct port isn't of interest, it likely contains various ideas that may be useful.

   \[1\] http://compilers.cs.uni-saarland.de/papers/lkh15\_cgo.pdf

   \[2\] https://anydsl.github.io/

4. Alex Elsayed says:[4 November 2015 8:22 AM](#74278b0de4ad4f1ad25d8e16f856622163ecd827)

   @John Cowan - the issue with not having a long-term perf story is that then everyone else writes their own \_short-term\_ perf story, which is "shoveling C into the extension furnace."

   The problem with that is it sacrifices a huge portion of what makes Python great, and kicks maintainability in the teeth to a nontrivial degree. Writing extensions in Rust seems to be helping some, but it's still not as clean as a good story in Python itself.

5. Pachi says:[4 November 2015 9:38 AM](#287a29b2a9aacaa2bcab316ed01c938fab4b1228)

   There's some activity around optimizing the CPython code generation these days. Have a look to the ideas in FAT Python here: http://article.gmane.org/gmane.comp.python.devel/155300

6. [John Cowan](http://www.ccil.org/~cowan) says:[4 November 2015 9:57 AM](#fe9f9e6c89acc112e27eaa98e41135346f5c53c1)

   I don't disagree at all. I'm just pointing out that, as is usually the case, psychology (or sociology) trumps technical factors. The 20th, hidden item in the Zen of Python is "Simple is better than fast", and that works only because of a general agreement not to say "Nyah, nyah, your precious language runs like a pig." Fortran has mostly outgrown this reputation, and Lisp would if people would bother to look at the facts (but they won't). The pro-Lisp bigotry of many Lispers, particularly old-school ones, is a response to general anti-Lisp bigotry, and as such comes under the head of internalized oppression.

7. [John Cowan](http://www.ccil.org/~cowan) says:[4 November 2015 10:28 AM](#5eabe9dece7a760bf571aab60b6f9229620d0ede)

   In short, while I wish FAT Python well, I think its chance of becoming part of the Python mainstream is None. I don't mean to imply that all of Lisp's problems are external, either: the big internal one is that Lisp is so flexible that [technical problems in other languages are social ones in Lisp.](http://www.winestockwebdesign.com/Essays/Lisp_Curse.html)

8. solrize says:[5 November 2015 11:47 AM](#8754c94ed6328821ed25e88ea0ac494108ffdab9)

   Do you also look at LuaJIT? Of course it's written in C but maybe some of its approaches can be adopted to Scheme.

   I also think the Erlang virtual machine BEAM is very interesting for its approach to concurrency, and it has a primitive native code compiler too, iirc.

   I agree that it's not necessary to go all-out for speed in an extension language. Debugging ease is important and I'd probably take it over speed. And if Guile is to be a full-blown implementation language instead of extension, sooner or later the Boehm GC will be the bottleneck. I'd like to see fast precise GC for the sake of GC-happy functional language front ends, but of course that breaks away from simple C integration. OTOH, I hacked on Emacs Lisp a fair amount and stuff like GCPRO never bothered me that much.

9. [Wilfred](http://www.wilfred.me.uk) says:[7 November 2015 2:59 PM](#69ed63979f2c5e9f496133310a6036675aa7755b)

   \> I don't know of good examples of AOT compilers that later added adaptive optimization.

   I've wondered about that, and wondered if it's an indictment of adaptive optimisation.

   In theory, adaptive optimisation can outperform AOT compilation due to its information on the current workload. However, there is an unavoidable bookkeeping overhead that seems to make AOT very hard to beat.

   After implementing AOT compilation, it seems more common for language implementations to build profile guided optimisations. I suppose it still requires the same bookkeeping, but you only have to run the executable once on a representative workload.

   I wish I had some hard numbers or papers I could cite, but I'm not aware of any. Outside of superoptimisation research, people seem to focus on incremental improvement more often than asking what the fastest possible performance is.

10. Ewan says:[8 November 2015 11:36 AM](#c6fd4d807bec1693dc2a753fd944eb43724c4d96)

    @wingo

    Great post as usual!

    @alex

    Are you using rust as a language for extending python? You say its helping. How are you going about this? Using the cpython api ot cffi? Or do you mean you are extending other programs using rust instead of python?

    In any event I think keeping a long term performance story is risky and dangerous. All the experience gained by our esteemed blog author would not have happened as there would be a continual push to the one true performance plan instead of the attempts that made real progress. The attemps that were deleted have real value in the experience gained. So there is this risk of not learning through smaller projects building through smaller victories towards an eventual goal which was planned out when there was little or no experience on hand.

    There is very real danger as well: if work is rejected because its too short term is then there is a risk that everyone puts their tools down and folds their arms until this legend of good performance exists. Then the community dries up and keels over.

    In other words: in the long enough performance plan, we're all dead.

11. Alex Elsayed says:[9 November 2015 3:01 AM](#bfb2b3d8c2e0b6385696ec4c6981de0f8d9bb3b5)

    @Ewan: I personally am not, but there have been a pretty large number of tutorials and "here's how I learned" sequences on doing exactly that, many filled with praise along the lines of \*oh thank goodness I don't have to deal with C, the lack of safety nets was nerve-wracking and error-prone.\*

    Most have been using cffi/ctypes, and in Ruby-land there's skylight.io which is using a Rust-based Ruby module as a core part of their product.

    The main benefit there, as I see it, is that using Rust for extensions gives up \*less\* of the nice nature of the language being extend (safety being a major element), but it's still Another Language With Different Semantics.

12. Dror Livne says:[24 November 2015 8:30 AM](#c49c4f6a98c48fb1ac71cae4e32c8a4ef55a5e98)

    Xavier Leroy has a somewhat related retrospective on the OCaml

    abstract-machine based implementation:

    http://cristal.inria.fr/~xleroy/talks/zam-kazam05.pdf

Comments are closed.
