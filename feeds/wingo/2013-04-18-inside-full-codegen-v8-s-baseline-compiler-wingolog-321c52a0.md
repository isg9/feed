---
title: inside full-codegen, v8's baseline compiler — wingolog
url: https://wingolog.org/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler
published: "2013-04-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler
---

# inside full-codegen, v8's baseline compiler — wingolog

## [inside full-codegen, v8's baseline compiler](/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler)

18 April 2013 3:45 PM

- [v8](/tags/v8)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)

Greetings to all. This is another nargish article on the internals of the V8 JavaScript engine. If that's your thing, read on. Otherwise, here's an [interesting interview with David Harvey](http://www.zcommunications.org/rebel-cities-and-urban-resistance-by-david-harvey). See you laters!

**full-codegen**

Today's topic is V8's baseline compiler, called "full-codegen" in the source. To recall, [V8 has two compilers](//wingolog.org/archives/2011/07/05/v8-a-tale-of-two-compilers): one that does a quick-and-dirty job (full-codegen), and one that focuses on hot spots (Crankshaft).

When you work with V8, most of the attention goes to Crankshaft, and rightly so: Crankshaft is how V8 makes code go fast. But everything goes through full-codegen first, so it's still useful to know how full-codegen works, and necessary if you go to hack on V8 itself.

The goal of this article is to recall the full-codegen to mind, so as to refresh our internal "efficiency model" of how V8 runs your code (cf. [Norvig's PAIP retrospective, #20](http://norvig.com/Lisp-retro.html)), and to place it in a context of other engines and possible models.

**stack machine**

All JavaScript code executed by V8 goes through full-codegen, so full-codegen has to be very fast. This means that it doesn't have time to do very much optimization. Even if it did have time to do optimization (which it doesn't), full-codegen lacks the means to do so. Since the code hasn't run yet when full-codegen compiles it, it doesn't know anything about types or shapes of objects. And as in other JS engines, full-codegen only compiles a function at a time, so it can't see very far. (This lets it compile functions lazily, the first time they are executed. Overall syntactic validity is ensured by a quick "pre-parse" over the source.)

I have to admit my surprise however at seeing again how simple full-codegen actually is: it's a stack machine! You don't get much simpler than that. It's shocking in some ways that this is the technology that kicked off the JS performance wars, back in 2008. The article I wrote a couple years ago does mention this, but since then I have spent a lot of time in JSC and forgot how simple things could be.

Full-codegen does have a couple of standard improvements over the basic stack machine model, as I mentioned in the "two compilers" article. One is that the compiler threads a "context" through the compilation process, so that if (say) `x++` is processed in "effect" context, it doesn't need to push the initial value of `x` on the stack as the result value. The other is that if the top-of-stack (ToS) value is cached in a register, so that even if we did need the result of `x++`, we could get it with a simple `mov` instruction. Besides these "effect" and "accumulator" (ToS) contexts, there are also contexts for pushing values on the stack, and for processing a value for "control" (as the test of a conditional branch).

**and that's it!**

That's the whole thing!

Still here?

What about type recording and all that fancy stuff, you say? Well, most of the fancy stuff is in crankshaft. Type recording happens in the inline caches that aren't really part of full-codegen proper.

Full-codegen does keep some counters on loops and function calls and such that it uses to determine when to tier up to Crankshaft, but that's not very interesting. (It is interesting to note that V8 used to determine what to optimizing using a statistical profiler, and that is no longer the case. It seems statistical profiling would be OK in the long run, but as far as I understand it didn't do a great job with really short-running code, and made it difficult to reproduce bugs.)

I think -- and here I'm just giving my impression, but fortunately I have readers that will call my bullshit -- I think that if you broke down basic JavaScript and ignored optimizable hot spots, an engine mostly does variable accesses, property accesses, and allocations. Those are the fundamental things to get right. In a browser you also have DOM operations, which [Blink](http://www.chromium.org/blink) will speed up at some point; and there are some domain-specific things like string operations and regular expressions of course. But ignoring that, there are just variables, properties, and allocations.

We'll start with the last bit. Allocation is a function of your choice of [value representation](//wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations), your use of "hidden classes" ("maps" in V8; all engines do the same thing now), and otherwise it's more a property of the runtime than of the compiler. (Crankshaft can lower allocation cost by using unboxed values and inlining allocations, but that's Crankshaft, not full-codegen.)

Property access is mostly handled by inline caches, which also handle method dispatch and relate to hidden classes.

The only thing left for full-codegen to do is to handle variable accesses: to determine where to store local variables, and how to get at them. In this case, again there's not much magic: either variables are really local and don't need to be looked up by name at runtime, in which case they can be allocated on the stack, or they persist for longer or in more complicated scopes, in which case they go on the heap. Of course if a variable is never referenced, it doesn't need to be allocated at all.

So in summary, full-codegen is quick, and it's dirty. It does its job with the minimum possible effort and gets out of the way, giving Crankshaft a chance to focus on the hot spots.

**comparison**

How does this architecture compare with what other JS engines are doing, you ask?

I have very little experience with SpiderMonkey, but as far as I can tell, with every release they they get closer and closer to V8's model. Earlier this month Kannan Vijayan wrote about [SpiderMonkey's new baseline compiler](http://blog.mozilla.org/javascript/2013/04/05/the-baseline-compiler-has-landed/), which looks to be exactly comparable to full-codegen. It's a [stack machine](http://hg.mozilla.org/mozilla-central/file/286cf3d33281/js/src/ion/BaselineCompiler.cpp#l916) that emits native code. It looks a little different because it operates on bytecode and not the AST, as SpiderMonkey still has an [interpreter](http://hg.mozilla.org/mozilla-central/file/286cf3d33281/js/src/jsinterp.cpp) hanging around.

JavaScriptCore, the WebKit JavaScript engine, is similar on the surface: it also has a baseline compiler and an optimizing compiler. (In fact it's getting another optimizing compiler soon, but that's a story for another article). But if you look deeper, I think JSC has diverged from V8's model in interesting ways.

Besides being a [register machine](//wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation), a model that has inherent advantages over a stack machine, JavaScriptCore also has a [low-level interpreter](//wingolog.org/archives/2012/06/27/inside-javascriptcores-low-level-interpreter) that uses the same calling convention as the optimizing compiler. I don't think JSC is moving in V8's direction in this regard; in fact, they could remove their baseline compiler entirely, relying instead on the interpreter and the optimizing compiler. JSC also has some neat tricks related to scoping (lazy tear-off); again, a topic for some future article.

**summary**

The field is still a bit open as to what is the best approach to use for cold code. It seems that an interpreter could still be a win, because if not, why would SpiderMonkey keep one around, now that they have a baseline compiler? And why would JSC add a new one?

At the same time, it seems to me that one could do more compile-time analysis at little extra cost, to speed up full-codegen by allocating more temporaries into slots; you would push and pop at compile-time to avoid doing it at run-time. It's unclear whether it would be worth it, though, as that analysis would have to run on all code instead of just the hot spots.

Hopefully someone will embark on the experiment; it should just be a simple matter of programming :) Until next time, happy hacking!

## related articles

- [generators in v8](/archives/2013/05/08/generators-in-v8)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)

### 9 responses

1. [Terry A. Davis](http://www.templeos.org) says:[18 April 2013 11:09 PM](#3861a9177475cd50f9b5da450b0b7f0c1df88b49)

   I'm happy to see a programmer help destroy the myth that one person cannot make a compiler. I briefly scanned your article. Maybe, you like me, maybe I'm like you. God is just. We'll never know. My enemy like to psy-op me. It kills time. Beats working. I would love to go to court. I'm in limbo. I killed a guy. No court. No market. Nothing. Limbo.

2. [Terry A. Davis](http://www.templeos.org) says:[18 April 2013 11:47 PM](#100327581a8c59274bdffbf8113aaf176dc960e2)

   No, you sound clueless and like crap and mixed in with bullshit lifted from me.

   Doesn't matter. No need to argue when I can bust-out Mr. God. Mr. God can strike you dead or say I tell the truth. Court would be nice.

   God says...

   22:9 Thou hast sent widows away empty, and the arms of the fatherless

   have been broken.

   22:10 Therefore snares are round about thee, and sudden fear troubleth

   thee; 22:11 Or darkness, that thou canst not see; and abundance of

   waters cover thee.

   22:12 Is not God in the height of heaven? and behold the height of the

   stars, how high they are! 22:13 And thou sayest, How doth God know?

   can he judge through the dark cloud? 22:14 Thick clouds are a

   covering to him, that he seeth not; and he walketh in the circuit of

   heaven.

   22:15 Hast thou marked the old way which wicked men have trodden?

   22:16 Which were cut down out of time, whose foundation was overflown

   with a flood: 22:17 Which said unto God, Depart from us: and what can

   the Almighty do for them? 22:18 Yet he filled their houses with good

   things: but the counsel of the wicked is far from me.

3. [Terry A Davis](http://www.templeos.org) says:[18 April 2013 11:56 PM](#8ea4c8b38e1d3c43bb664b9e3a8a4681553f4a18)

   Get sterilized and made impotent. Get humiliated.

   http://www.youtube.com/watch?v=yknkKIX4ikE

   I got locked up in mental hospitals over and over and put in restraints.

   I got patronized and tortured by psy-ops. They know what they did. I get spit on by all the fake people everywhere who are psy-ops.

   fake people. agents. westworld. Run them over for fun.

4. [Andy Wingo](http://wingolog.org/) says:[19 April 2013 8:20 AM](#ed8d2de63e93873663291ed87846db1f25d0aaa2)

   Hi Terry, thanks for stopping by; I hadn't realized that you renamed your OS; here's [an interesting discussion I found on that](http://www.metafilter.com/119424/An-Operating-System-for-Songs-from-God).

   Still, I should let you know that while on-topic comments are appreciated, I'll probably remove any off-topic ones. Thanks :)

5. kripken says:[20 April 2013 4:09 AM](#2b87c21cdde0b227c23f00f4c09d857e43008ae4)

   Note that there are other reasons for having an interpreter, than performance. For example, there are a lot of new language features coming into JavaScript - generators, modules, binary data, let, etc. etc. - and implementing such things in an interpreter is generally much simpler than a JIT, even a relatively non-optimizing JIT.

   Then later you add JIT support for those features to make them fast, when necessary, etc.

6. [Andy Wingo](http://wingolog.org/) says:[20 April 2013 9:34 AM](#0eb39374474ca3ed5abac25862db42283a5534b7)

   Hi kripken,

   It's true that getting optimizing JITs to deal with e.g. generators is tricky, but there is no significant problem getting a baseline JIT to support new features -- especially if you have a nice assembler abstraction like the one JSC has. The problems are more in the runtime; the whole point of a baseline JIT is to do the same thing your interpreter would do, not to optimize.

   I'm currently finishing up an implementation of generators for V8, and code generation has not been a difficulty. Just my 2¢ :)

7. Steven Roussey says:[23 April 2013 4:30 PM](#e62daa44f0ca73586c1982b5d17f1708144fc442)

   Another option is to use another thread to do optimizations, like IE10.

8. [Sanjoy Das](http://playingwithpointers.com) says:[12 May 2013 5:34 PM](#4e06df25f9cfc1175be92ab7110c0400d416eb4d)

   @Steven

   v8 has had experimental support for parallel optimization (running the heavy parts of Crankshaft on a second thread) for around a year now. Have a look at --parallel\_recompilation.

9. rahul prakash says:[8 September 2015 9:42 AM](#570111bcf399e721a0958dbd26b0edc486d63705)

   Hi I am unable to understand the usage and working of V8 properly. Even unable to find proper examples to understand v8 properties and its functionality like Context, persistentcontext. Can anybody will help me out from it.

   Regards

   Rahul Prakash

Comments are closed.
