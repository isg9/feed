---
title: JavaScriptCore, the WebKit JS implementation — wingolog
url: https://wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation
published: "2011-10-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation
---

# JavaScriptCore, the WebKit JS implementation — wingolog

## [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)

28 October 2011 3:51 PM

- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)
- [javascriptcore](/tags/javascriptcore)
- [igalia](/tags/igalia)
- [jsc](/tags/jsc)
- [v8](/tags/v8)
- [scheme](/tags/scheme)
- [guile](/tags/guile)

My readers will know that I have recently had the pleasure of looking into the [V8](//wingolog.org/tags/v8) JavaScript implementation, from [Google](http://code.google.com/p/v8). I'm part of a small group in [Igalia](http://www.igalia.com/) doing compiler work, and it's clear that in addition to being lots of fun, JavaScript implementations are an important part of the compiler market today.

But V8 is not the only JS implementation in town. Besides Mozilla's [SpiderMonkey](https://developer.mozilla.org/en/SpiderMonkey), which you probably know, there is another major Free Software JS implementation that you might not have even heard of, at least not by its proper name: JavaScriptCore.

**jsc: js for webkit**

JavaScriptCore (JSC) is the JavaScript implementation of the [WebKit](http://webkit.org/) project.

In the beginning, JavaScriptCore was a simple tree-based interpreter, as Mozilla's SpiderMonkey was. But then in June of 2008, a few intrepid hackers at Apple wrote a compiler and bytecode interpreter for JSC, threw away the tree-based interpreter, and called the thing [SquirrelFish](http://www.webkit.org/blog/189/announcing-squirrelfish/). This was eventually marketed as "Nitro" inside Apple's products\[0\].

JSC's bytecode interpreter was great, and is still pretty interesting. I'll go into some more details later in this article, because its structure conditions the rest of the implementation.

But let me continue here with this historical sketch by noting that later in 2008, the WebKit folks added inline caches, a regular expression JIT, and a simple method JIT, and then called the thing [SquirrelFish Extreme](http://www.webkit.org/blog/214/introducing-squirrelfish-extreme/). Marketers called this Nitro Extreme. (Still, the proper name of the engine is JavaScriptCore; [Wikipedia currently gets this one wrong.](http://en.wikipedia.org/wiki/JavaScript_engine))

One thing to note here is that the JSC folks were doing great, well-factored work. It was so good that SpiderMonkey hackers at Mozilla adopted JSC's regexp JIT compiler and their native-code assembler directly.

As far as I can tell, for JSC, 2009 and 2010 were spent in "consolidation". By that I mean that JSC had a JIT and a bytecode interpreter, and they wanted to maintain them both, and so there was a lot of refactoring and tweaking to make them interoperate. This phase consolidated the SFX gains on x86 architectures, while also adding implementations for ARM and other architectures.

But with the release of V8's Crankshaft in late 2010, the JS performance bar had been lowered again (assuming it is a limbo), and so JSC folks started working on what they call their "DFG JIT" (DFG for "data-flow graph"), which aims be more like Crankshaft, basically.

It's possible to configure a JSC with all three engines: the interpreter, the simple method JIT, and the DFG JIT. In that case there is tiered compilation between the three forms: initial parsing and compilation produces bytecode, that can be optimized with the method JIT, that can be optimized by the DFG JIT. In practice, though, on most platforms the interpreter is not included, so that all code runs through the method JIT. As far as I can tell, the DFG JIT is shipping in Mac OS X Lion's Safari browser, but it is not currently enabled on any platform other than 64-bit Mac. (I am working on getting that fixed.)

**a register vm**

The interpreter has a number of interesting pieces, but it is important mostly for defining the format of bytecode. Bytecode is effectively the high-level intermediate representation (IR) of JSC.

To put that into perspective, in V8, the high-level intermediate representation is the JS source code itself. When V8 first sees a piece of code, it *pre-parses* it to raise early syntax errors. Later when it needs to analyze the source code, either for the full-codegen compiler or for Hydrogen, it re-parses it to an AST, and then works on the AST.

In contrast, in JSC, when code is first seen, it is fully parsed to an AST and then that AST is compiled to bytecode. After producing the bytecode, the source text isn't needed any more, and so it is forgotten. The interpreter interprets the bytecode directly. The simple method JIT compiles the bytecode directly. The DFG JIT has to re-parse the bytecode into an SSA-style IR before optimizing and producing native code, which is a bit more expensive but worth it for hot code.

As you can see, bytecode is the common language spoken by all of JSC's engines, so it's important to understand it.

Before really getting into things, I should make an aside about terminology here. Traditionally, at least in my limited experience, a virtual machine was considered to be a piece of software that interprets sequences of virtual instructions. This would be in contrast to a "real" machine, that interprets sequences of "machine" or "native" instructions in hardware.

But these days things are more complicated. A common statement a few years ago would be, "is JavaScript interpreted or compiled?" This question is nonsensical, because "interpreted" or "compiled" are properties of implementations, not languages. Furthermore the implementation can compile to bytecode, but then interpret that bytecode, as JSC used to do.

And in the end, if you compile all the bytecode that you see, where is the "virtual machine"? V8 hackers still call themselves "virtual machine engineers", even as there is no interpreter in the V8 sources (not counting the ARM simulator; and what of a program run under qemu?).

All in all though, it is still fair to say that JavaScriptCore's high-level intermediate language is a sequence of virtual instructions for an abstract register machine, of which the interpreter and the simple method JIT are implementations.

When I say "register machine", I mean that in contrast to a "stack machine". The difference is that in a register machine, all temporary values have names, and are stored in slots in the stack frame, whereas in a stack machine, temporary results are typically pushed on the stack, and most instructions take their operands by popping values off the stack.

(Incidentally, V8's full-codegen compiler operates on the AST in a stack-machine-like way. Accurately modelling the state of the stack when switching from full-codegen to Crankshaft has been a source of many bugs in V8.)

Let me say that for an interpreter, I am totally convinced that register machines are the way to go. I say this as a Guile co-maintainer, which has a stack VM. Here are some reasons.

First, stack machines penalize named temporaries. For example, consider the following code:

```
(lambda (x)
  (* (+ x 2)
     (+ x 2)))

```

We could do common-subexpression elimination to optimize this:

```
(lambda (x)
  (let ((y (+ x 2)))
    (* y y)))

```

But in a stack machine is this really a win? Consider the sequence of instructions in the first case:

```
; stack machine, unoptimized
0: local-ref 0      ; x
1: make-int8 2
2: add
3: local-ref 0      ; x
4: make-int8 2
5: add
6: mul
7: return

```

Contrast this to the instructions for the second case:

```
; stack machine, optimized
0: local-ref 0      ; push x
1: make-int8 2      ; push 2
2: add              ; pop x and 2, add, and push sum
3: local-set 1      ; pop and set y
4: local-ref 1      ; push y
5: local-ref 1      ; push y
6: mul              ; pop y and y, multiply, and push product
7: return           ; pop and return

```

In this case we really didn't gain anything, because the storing values to locals and loading them back to the stack take up separate instructions, and in general the time spent in a procedure is linear in the number of instructions executed in the procedure.

In a register machine, on the other hand, things are easier, and CSE is definitely a win:

```
0: add 1 0 0           ; add x to x and store in y
1: mul 2 1 1           ; multiply y and y and store in z
2: return 2            ; return z

```

In a register machine, there is no penalty to naming a value. Using a register machine reduces the push/pop noise around the instructions that do the real work.

Also, because they include the names (or rather, locations) of their operands within the instruction, register machines also take fewer instructions to do the job. This reduces dispatch cost.

In addition, with a register VM, you know the size of a call frame before going into it, so you can avoid overflow checks when pushing values in the function. (Some stack machines also have this property, like the JVM.)

But the big advantage of targeting a register machine is that you can take advantage of traditional compiler optimizations like CSE and register allocation. In this particular example, we have used three virtual registers, but in reality we only need one. The resulting code is also closer to what real machines expect, and so is easier to JIT.

On the down side, instructions for a register machine typically occupy more memory than instructions for a stack machine. This is particularly the case for JSC, in which the opcode and each of the operands takes up an entire machine word. This was done to implement "direct threading", in which the opcodes are not indexes into jump tables, but actually are the addresses of the corresponding labels. This might be an acceptable strategy for an implementation of JS that doesn't serialize bytecode out to disk, but for anything else the relocations are likely to make it a lose. In fact I'm not sure that it's a win for JSC even, and perhaps the bloat was enough of a lose that the interpreter was turned off by default.

Stack frames for the interpreter consist of a six-word frame, the arguments to the procedure, and then the locals. Calling a procedure reserves space for a stack frame and then pushes the arguments on the stack -- or rather, sets them to the last n \+ 6 registers in the stack frame -- then slides up the frame pointer. For some reason in JSC the stack is called the "register file", and the frame pointer is the "register window". Go figure; I suppose the names are as inscrutable as the "activation records" of the stack world.

**jit: a jit, a method jit**

I mention all these details about the interpreter and the stack (I mean, the register file), because they apply directly to the method JIT. The simple method JIT (which has no name) does the exact same things that the bytecode interpreter does, but it does them via emitted machine instructions instead of interpreting virtual instructions.

There's not much to say here; jitting the code has the result you would expect, reducing dispatching overhead, while at the same time allowing some context-specific compilation, like when you add a constant integer to a variable. This JIT is really quick-and-dirty though, so you don't get a lot of the visibility benefits traditionally associated with method JITs like what HotSpot's C1 or C2 currently have. Granted, the register VM bytecode does allow for some important optimizations to happen, but JSC currently doesn't do very much in the way of optimizing bytecode, as far as I can tell.

Thinking more on the subject, I suspect that for Javascript, CSE isn't even possible unless you know the types, as a `valueOf()` callback could have side effects.

**an interlude of snarky footnotes**

Hello, reader! This is a long article, and it's a bit dense. I had some snarky footnotes that I enjoyed writing, but it felt wrong to put them at the end, so I thought it better to liven things up in the middle here. The article continues in the next section.

0\. In case you didn't know, compilers are approximately 37% composed of marketing, and rebranding is one of the few things you can do to a compiler, marketing-wise, hence the name train: SquirrelFish, Nitro, SFX, Nitro Extreme...\[1\] As in, in response to "I heard that Nitro is slow.", one hears, "Oh, man they totally fixed that in SquirrelFish Extreme. It's blazingly fast!\[2\]"

1\. I don't mean to pick on JSC folks here. V8 definitely has this too, with their "big reveals". Firefox people continue to do this for some reason (SpiderMonkey, TraceMonkey, JaegerMonkey, IonMonkey). I expect that even they have forgotten the reason at this point. In fact the JSC marketing has lately been more notable in its absence, resulting in a dearth of useful communication. At this point, in response to "Oh, man they totally are doing a great job in JavaScriptCore", you're most likely to hear, "JavaScriptCore? Never heard of it. Kids these days hack the darndest things."

2\. This is the other implement in the marketer's toolbox: "blazingly fast". It means, "I know that you don't understand anything I'm saying, but I would like for you to repeat this phrase to your colleagues please." As in, "LLVM does advanced TBAA on the SSA IR, allowing CSE and LICM while propagating copies to enable SIMD loop vectorization. It is blazingly fast."

**dfg: a new crankshaft for jsc?**

JavaScriptCore's data flow graph (DFG) JIT is work by Gavin Barraclough and Filip Pizlo to enable speculative optimizations for JSC. For example, if you see the following code in JS:

```
a[i++] = 0.7*x;

```

then `a` is probably an array of floating-point numbers, and `i` is probably an integer. To get great performance, you want to use native array and integer operations, so you speculatively compile a version of your code that makes these assumptions. If the assumptions don't work out, then you bail out and try again with the normal method JIT.

The fact that the interpreter and simple method JIT have a clear semantic model in the form of bytecode execution makes it easy to bail out: you just reconstruct the state of the virtual registers and register window, then jump back into the code. (V8 calls this process "deoptimization"; the DFG calls it "speculation failure".)

You can go the other way as well, switching from the simple JIT to the optimized DFG JIT, using [on-stack replacement](//wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8). The DFG JIT does do OSR. I hear that it's needed if you want to win [Kraken](http://krakenbenchmark.mozilla.org/), which puts you in lots of tight loops that you need to be able to optimize without relying on being able to switch to optimized code only on function re-entry.

When the DFG JIT is enabled, the interpreter (if present) and the simple method JIT are augmented with profiling information, to record what types flow through the various parts of the code. If a loop is executed a lot (currently more than 1000 times), or a function is called a lot (currently about 70 times), the DFG JIT kicks in. It parses the bytecode of a function into an SSA-like representation, doing inlining and collecting type feedback along the way. This probably sounds [very familiar to my readers](//wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler).

The difference between JSC and Crankshaft here is that Crankshaft parses out type feedback from the inline caches directly, instead of relying on in-code instrumentation. I think Crankshaft's approach is a bit more elegant, but it is prone to lossage when GC blows the caches away, and in any case either way gets the job done.

I mentioned inlining before, but I want to make sure that you noticed it: the DFG JIT does do inlining, and does so at parse-time, like HotSpot does. The type profiling (they call it "value profiling") combined with some cheap static analysis also allows the DFG to unbox int32 and double-precision values.

One thing that the DFG JIT doesn't do, currently, is much in the way of code motion. It does do some dead-code elimination and common-subexpression elimination, and as I mentioned before, you need the DFG's value profiles in order to be able to do this correctly. But it does not do loop-invariant code motion.

Also, the DFG's register allocator is not as good as Crankshaft's, yet. It is hampered in this regard by the JSC assembler that I praised earlier; while indeed a well-factored, robust piece of code, JSC's assembler has a two-address interface instead of a three-address interface. This means that instead of having methods like `add(dest, op1, op2)`, it has methods like `add(op1, op2)`, where the operation implicitly stores its result in its first operand. Though it does correspond to the x86 instruction set, this sort of interface is not great for systems where you have more registers (like on x86-64), and forces the compiler to shuffle registers around a lot.

The counter-based optimization triggers do require some code to run that isn't strictly necessary for the computation of the results, but this strategy does have the nice property that the DFG performance is fairly predictable, and measurable. Crankshaft, on the other hand, being triggered by a statistical profiler, has [statistically variable performance](http://arewefastyet.com/?a=b&machine=8).

And speaking of performance, [AWFY on the mac](http://arewefastyet.com/?a=b&machine=8) is really where it's at for JSC right now. Since the DFG is only enabled by default on recent Mac OS 64-bit builds, you need to be sure you're benchmarking the right thing.

Looking at the results, I think we can say that JSC's performance on the V8 benchmark is really good. Also it's interesting to see JSC beat V8 on SunSpider. Of course, there are lots of quibbles to be had as to the validity of the various benchmarks, and it's also clear that V8 is the fastest right now once it has time to warm up. But I think we can say that JSC is doing good work right now, and getting better over time.

**future**

So that's JavaScriptCore. The team -- three people, really -- is mostly focusing on getting the DFG JIT working well right now, and I suspect they'll be on that for a few months. But we should at least get to the point where the DFG JIT is working and enabled by default on free systems within a week or two.

The one other thing that's in the works for JSC is a new generational garbage collector. This is progressing, but slowly. There are stubs in the code for card-marking write barriers, but currently there is no such GC implementation, as far as I can tell. I suspect that someone has a patch they're cooking in private; we'll see. At least JSC does have a [Handle](http://trac.webkit.org/browser/trunk/Source/JavaScriptCore/heap/Handle.h) API, unlike SpiderMonkey.

**conclusion**

So, yes, in summary, JavaScriptCore is a fine JS implementation. Besides being a correct implementation for real-world JS -- something that is already saying quite a lot -- it also has good startup speed, is fairly robust, and is working on getting an optimizing compiler. There's work to do on it, as with all JS implementations, but it's doing fine.

Thanks for reading, if you got this far, though I understand if you skipped some parts in the middle. Comments and corrections are most welcome, from those of you that actually read all the way through, of course :). Happy hacking!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)
- [inside javascriptcore's low-level interpreter](/archives/2012/06/27/inside-javascriptcores-low-level-interpreter)
- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [eval, that spectral hound](/archives/2012/02/01/eval-that-spectral-hound)
- [value representation in javascript implementations](/archives/2011/05/18/value-representation-in-javascript-implementations)

### 26 responses

1. Jasper St. Pierre says:[28 October 2011 5:17 PM](#a1e28064c2db80849c2695a8835c23f060246dc2)

   (still reading, it's an interesting article that will take me much longer than I have now to go through. Bookmarked!)

   "After producing the bytecode, the source text isn't needed any more, and so it is forgotten."

   So does Function.prototype.toString() just invent something up based on the bytecode?

2. [wingo](http://wingolog.org/) says:[28 October 2011 6:06 PM](#760944818a851b3947a02afd95121fb7119402ee)

   Thanks for the note, Jasper; that's probably a misstatement. JS requires you to keep the source around. But the source doesn't serve the same role to JSC's compiler as it does to V8's compiler.

3. [Jeff Walden](http://whereswalden.com/) says:[28 October 2011 6:53 PM](#484f3f58f1b3c2f2d3e49c0efea19f2caaafb77f)

   "In the beginning, JavaScriptCore was a simple tree-based interpreter, as Mozilla's SpiderMonkey was."

   If by this you mean to say SpiderMonkey executed code by walking a parse tree, this is not the case. As far as I know, SpiderMonkey has always generated and executed bytecode. See for example [jsemit.c](http://mxr.mozilla.org/classic/source/js/src/jsemit.c) from the Netscape 4 source dump from 1998. These days, of course, we do less bytecode interpreting and more JITted execution of bytecode-compiled-to-native-code. But we still do some bytecode interpreting even now, because JIT compilation, even when done as fast as possible, is still slower than bytecode interpretation for code run only a couple times.

4. [Jeff Walden](http://whereswalden.com/) says:[28 October 2011 7:25 PM](#7acc92cfc3c8c7e91554b7296e82745ecc9422fd)

   Also, "stratey".

   As I understand JSC keeps the original source code around for toString-based decompilation. This is probably the right approach: simple, straightforward, entirely non-lossy. As far as I know, every engine but SpiderMonkey does this. (SpiderMonkey actually decompiles SpiderMonkey bytecode, with some annotations, to JS code. This works surprisingly well, but historically it has been a large source of bugs. It also imposes considerable cost on changing the bytecode format, because any bytecode that's generated must also be decompilable.)

5. Luke Wagner says:[29 October 2011 1:00 AM](#799dd8fa931464b9f2aca79edb94b32e07887eea)

   "Firefox people continue to do this for some reason (SpiderMonkey, TraceMonkey, JaegerMonkey, IonMonkey). I expect that even they have forgotten the reason at this point."

   These are not marketing names, they are just the project names used by JS hackers. Project names are useful and enjoyable for those involved. The reason anyone has heard of them is because Mozilla does all its development in the open so anyone can see the bugs flying by or developer blog posts. This is the exact opposite of the "big reveals" you mentioned.

6. gasche says:[29 October 2011 6:58 AM](#90da6a6c1f5ed295991fc8e4ba1df4d8cdfa8b84)

   \> For some reason in JSC the stack is called

   \> the "register file", and the frame pointer

   \> is the "register window". Go figure;

   There is a hardware concept of "register window" in some architectures that applies quite well to (small enough) stack frames in a register machine:

   http://en.wikipedia.org/wiki/Register\_window

7. [wingo](http://wingolog.org/) says:[29 October 2011 9:17 AM](#43e2d7b73b428d78e9d48a29f144e05519c0b322)

   Thanks for the corrections, Jeff. I didn't know that SpiderMonkey actually decompiled their bytecode to make a function's toString() value.

   Luke, it was a friendly jab at Mozilla folk. I haven't felt the need to write about SpiderMonkey because you all do it so well already. I do think that the profusion of names is getting a bit confusing, though.

   Regarding your point on V8, it's true that V8 folks don't plan in the open, but they do review and commit in the open. There haven't been any big "code drops" for a year or so, unless you count Dart of course.

   Gasche, thanks for the note (and continued kind words, here and in other places). I am familiar with the concept but it seems a little silly to apply to values on a stack. Also, I have the impression that register windows are not in favor any more as a hardware implementation technique; I seem to recall a paper (was it Ungar's 1993 thesis on Self?) that dealt with the topic extensively.

   I'm headed out for a long weekend in the country; catch you all when I get back!

8. azakai says:[29 October 2011 5:51 PM](#897b1a7bc3781d6855153d6c93a1b5d55e5f2252)

   \> it's also clear that V8 is the fastest right now once it has time to warm up

   Perhaps on the v8 and kraken benchmarks, but just barely on kraken, and much of the v8 difference is due to GC (which has little to do with warming up). Also, looking at individual benchmarks inside the v8 and kraken suites, each JS engine wins on several of them.

   And on other benchmarks, v8 is often slower. As just one example, on Fabrice Bellard's CPU emulator running linux, Chrome is about half as fast as Firefox.

   We have great competition now in the JS engine space, no single engine is a clear leader.

   \> Regarding your point on V8, it's true that V8 folks don't plan in the open, but they do review and commit in the open. There haven't been any big "code drops" for a year or so, unless you count Dart of course.

   All the major upgrades to V8 have been code drops: The initial version of V8, and CrankShaft. And given the progress in SpiderMonkey and JavaScriptCore, it is a safe bet Google is working on another code drop in secret. To assume otherwise means Google is ok with dropping behind in the JS speed race in several months' time, which seems very unlikely.

9. [Jeff Walden](http://whereswalden.com/) says:[29 October 2011 9:37 PM](#0d4dc418656814445fee80d00edaf59d09654a22)

   Further to the SpiderMonkey naming bit, I am reminded of Patrick Winston's intro AI lectures that I attended at MIT. In each lecture he would note a "gold-star idea". (Corny name, and we'd joke about it a bit, but the key concepts were worth highlighting.)

   One of them, as best as I recall, was that "names give you power". The fading (since when? but it does seem to be, now) term "Ajax" is an example of this. Before its introduction people were doing what it meant, but there wasn't an easy way for them to \*say\* what they meant. After, the hook the term provided made it easy to discuss such things. The "Ajax" name gave developers power in discussing, proposing, analyzing, and creating interactive websites.

   Apply this to SpiderMonkey, and you can see that the names for the JITs help people talk about them more easily. But it's not just a marketing thing: developers and Mozilla project planners (as well as other community onlookers) find the TraceMonkey, JaegerMonkey, IonMonkey names useful. So when deciding, say, when to generally enable a particular JIT in Mozilla code, planning could talk about "enabling JaegerMonkey" rather than "enabling the methodjit", and instead of hearing engineering lingo that fades into background noise of a technical meeting, you hear a distinctive name which is more easily mentally grasped.

   That's much of what good names do: give better hooks for thinking about something. The alternatives -- tracejit, methodjit, ??? (maybe you see the power of a name for this one, because I don't know what we'd pithily call it) -- are perhaps more precise. But the \*Monkey names are more distinctive, and arguably more evocative.

   Also, as Luke notes, the monkey names are just more fun. :-)

10. Justo says:[29 October 2011 9:54 PM](#6545ddca46a01201f46420b1d7cdfdc4ece757d3)

    azakai: I'm not sure how closely you follow v8 development, but wingo is correct.

    While there hasn't been a recent fundamental change as big as crankshaft landed in v8, v8 today is considerably faster than it was when crankshaft was dropped, and the type specialization and IR functionality that crankshaft introduced is better and more widely used. Each review and commit has indeed been public. The next really big development is the new GC (talked about at jsconf.eu), which anyone who has cared to has been able to watch in development in a v8 branch since it started, and discuss it with the team on the mailing list.

    Admittedly, the node community is a little more obsessive in following these things than most, but the v8 team has always been very open to us. It's actually the "big reveal" blog posts and documentation drops coming from the (very busy) Spidermonkey team that I find harder to feel up to date on :)

11. azakai says:[30 October 2011 5:39 AM](#06d23deaa16e71e6ad95f850c2b087043d526db8)

    @Justo

    I don't think there is any contradiction between what we both said. To summarize:

    1\. Major V8 upgrades have always been developed in secret and then code dropped.

    2\. Minor upgrades - which can accumulate into significant speedups, for sure - are done in the open.

    3\. It would be extremely surprising if the V8 team is not working on the next major upgrade in secret right now, because otherwise they are at high risk of falling behind in several month's time - it's entirely unreasonable to assume Google is ok with that.

    4\. The GC upgrade has been done in the open, but I don't consider it a major engine upgrade. In particular it will only affect a few parts of only one benchmark. (And also, the fact it has been done in the open indicates Google doesn't consider it important enough to do in secret as it does other major upgrades.)

    The V8 team is of course not doing anything horrible by developing in secret. It's their right to develop however they want. But many in the open source community dislike code drops, and I see no indication the V8 team is moving away from doing them, despite the comment I was responding to.

12. Justo says:[30 October 2011 6:27 AM](#c1c213a46461747356f4a9f6f08fab97a9562166)

    Yes, begging the question usually does go well, especially when you have your narrative pre-constructed.

    You can be proud of your work for its own sake and leave it at that. The undertone here is grating.

13. azakai says:[30 October 2011 6:55 AM](#b636100bca5e41542292fb7e331ffe5a288dbf75)

    I am not sure what undertone you mean. But to clarify my position: I am a big fan of the V8 project. They have done many awesome things, that have driven other JS engines to adopt their approaches or to develop other approaches in order to compete. Overall the JS landscape would be much poorer without V8. I have also filed various bugs on V8, when I found issues, to do my part to help out. I have also embedded V8 in a project of mine in the past (a game engine).

    So I am very positive on V8. And I have the utmost respect for the V8 team.

    All I was saying above, is that V8 has a history of code drops. And that to assume there is no code drop in the works now, is very unlikely, given the competitive situation and that Google will not take the risk of dropping behind (is there any doubt of that? It would be an insult to Google to assume otherwise). And note that I did clarify that code drops are not wrong, so I am not even saying that the code drop approach is wrong (although I myself prefer open development). I was only responding above to a comment that I understood to imply V8 was leaving off of doing code drops.

    I hope that clarifies what I was saying. I meant no offense to V8 or to you, and I apologize for any miscommunication.

14. azakai says:[30 October 2011 7:10 AM](#18329570ed1e029073408a8250933cf2e703f6c7)

    To further clarify, my logic is this:

    1\. Nothing in the public development on V8 looks like it will push performance forward \*dramatically\* (although there is incremental progress, for sure).

    2\. Other JS engines have publicly-known projects in the works that \*will\* push their performance forward dramatically.

    3\. Google, we can assume, is \*not\* ok with dropping behind in the JS speed race.

    4\. V8 has a history of code drops.

    So the most likely conclusion seems to be that V8 has another major upgrade being worked on in secret. Unless I am wrong in one of the assumptions 1-4: Please correct me if so,

    Again, I am not saying there is anything wrong with anything here. I have no negativity on V8 - as I said above, I love the project.

    In fact, to assume there is no secret code drop in the works there would be to say something negative about V8 - that they have no plan for a big leap forward in speed (since I see no public plans or work of that nature). I \*do\* have faith in them that they do have such plans, and the capability to implement them :)

15. [Tom Schuster](http://javascript-reverse.tumblr.com) says:[30 October 2011 1:01 PM](#295272320f8c580a2a84520fe0dfc935f6bd5d9d)

    Also one think to keep in mind, it is not necessary the decision of the v8 team to make these code drops.

    The v8 people are in fact very cool people.

    Google is not a pure open-source company.

16. azakai says:[30 October 2011 4:02 PM](#abb3813f4fe5cd75a88a9a10b102998bab65168f)

    \> Also one think to keep in mind, it is not necessary the decision of the v8 team to make these code drops.

    The v8 people are in fact very cool people.

    Google is not a pure open-source company.

    Good point, I agree. All the Google engineers I have met are indeed very cool, code drops might be a corporate thing.

17. [Eric](http://norstrulde.org) says:[31 October 2011 1:53 AM](#cfb7c2b2a28418aab0a2dca3cbe320352b5baea9)

    Great post. Thanks wingo.

18. Andy Tai says:[31 October 2011 8:43 PM](#049811e4fa48e1a665fc4d823f21f587509a593c)

    So what does the observation here mean for Guile, and similar technology in the GNU project (such as libjit)?

19. Ossk says:[1 November 2011 11:59 AM](#92d09bc91ed772a59c412e5e903b9f5bfcd9416a)

    Hoping the gnome-shell will be ported to this here :O

20. Verte says:[2 November 2011 3:48 AM](#1a1a311140157fd76829dadd97f41d12819122ab)

    \> The DFG JIT does do OSR. I hear that it's needed if you want to

    \> win Kraken, which puts you in lots of tight loops that you need

    \> to be able to optimize without relying on being able to switch

    \> to optimized code only on function re-entry.

    What we do (which sure sounds simpler) is switch to optimised code on loop re-entry.

21. something says:[3 November 2011 6:07 PM](#c2d0bea9e39d67321560dc6b1219b6142ee50dba)

    you sir, are intense

22. azakai says:[11 November 2011 4:05 AM](#83b98bb24713e7cbcf62581db8e5a2cfa98b69bf)

    JavaScriptCore just got \*much\* faster on 32-bit,

    http://arewefastyet.com/

    :)

    I assume this means their DFG JIT landed?

23. [wingo](http://wingolog.org/) says:[11 November 2011 4:00 PM](#e5c4e40a927f31822483b388f3a584398909ad70)

    Azakai, indeed :-)

    See bugs [71686](https://bugs.webkit.org/show_bug.cgi?id=71686) and [71373](https://bugs.webkit.org/show_bug.cgi?id=71373).

24. [Nicholas Shanks](http://web.nickshanks.com/) says:[15 November 2011 4:39 PM](#17a76b3c1448d4bb0b97d7ea760578ece9388490)

    Apparently WebKit has "more than 10%" of the global browser market:

    http://www.igalia.com/nc/work/project/item/webkitgtk/

    Perhaps you could pass a note to your company's web dev to update his figures? :)

25. [wingo](http://wingolog.org/) says:[16 November 2011 11:40 AM](#155b8cc2e5cc5ea69e0a4ce597b29146f533dd94)

    Thanks for the note Nicholas, will do. Like any web site run by programmers, it is in need of a lot of love ;-)

26. Wilson L says:[2 October 2013 6:39 PM](#664e7d9a85b142cb4b9a25c1437c845e9c8b7564)

    Any updates on which platforms the DFG JIT has been enabled for? Any chance we'll see it on iOS anytime soon?

Comments are closed.
