---
title: ecmascript generators from a performance perspective — wingolog
url: https://wingolog.org/archives/2013/06/11/ecmascript-generators-from-a-performance-perspective
published: "2013-06-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/06/11/ecmascript-generators-from-a-performance-perspective
---

# ecmascript generators from a performance perspective — wingolog

## [ecmascript generators from a performance perspective](/archives/2013/06/11/ecmascript-generators-from-a-performance-perspective)

11 June 2013 1:48 PM

- [ecmascript](/tags/ecmascript)
- [es6](/tags/es6)
- [igalia](/tags/igalia)
- [v8](/tags/v8)
- [compilers](/tags/compilers)
- [partial evaluation](/tags/partial%20evaluation)
- [performance](/tags/performance)

It's been really gratifying to see the recent interest in [ES6 generators in V8](//wingolog.org/archives/2013/05/08/generators-in-v8). I'm still a Schemer at heart, but one thing all language communities should learn from JS is its enthusiasm and sense of joyful play. One can't spend much time around folks like [James Long](http://jlongster.com/) or [Domenic Denicola](https://twitter.com/domenic) without feeling the urge to go out and build something fun.

**perfie perf perf perf**

But this is a blog about nargery! A lot of people have been speculating about the performance impact of using generators, and I've been meaning to write about it, so perhaps this article will tickle your neuron.

A generator is a function that can suspend. So the first perf question to ask yourself is, does the performance of generators matter at all? If your generator spends more time suspended than active, then probably not. Though it's not the only application of generators, if you are using generators for asynchronous programming, then probably generator performance matters not at all.

But that's not very satisfying, right? Even though it probably doesn't matter, maybe it does, and then you need to make sure your mental model of what is fast and what is slow corresponds to reality.

**crankshaft**

So, the skinny: as implemented in V8, the primary perf impact of generators is that they do not get optimized by Crankshaft.

As you probably know, Crankshaft is V8's optimizing compiler. There are two ways a function can be optimized: on entry, because the function is called many times, or within a loop, because the loop is taking too long. In the first case it's fairly easy: you kick off a parallel task to recompile the function, reading the type feedback collected by the inline caches from the baseline compiler. Once you've made the optimized function, you swap the function's code pointer, and the next time you call the function you get the optimized version instead of the generic version.

The other way is if you are in a function, in a hot loop, and you decide to optimize the function while you are in it. This is called on-stack replacement (OSR), and is important for functions that are only called once but do have hot loops. (Cough KRAKEN cough.) It's trickier to do parallel compilation with OSR, as the function might have left the loop by the time recompilation finishes, but oh well.

So in summary, when V8 goes to optimize a function it will be in one of two states: *inactive*, because it hasn't started to run yet, or *active*, because it's stuck in a tight loop.

Generators introduce a new state for functions: *suspended*. You can know whether a function is active or inactive, but you can't know how many suspended generator activations are out in the JS heap. So you have multiple potential entry points to the function, which complicates the optimizer as it has to work on graphs with irreducible loops. Perhaps it's still possible to optimize, as OSR is a form of irreducible loops; I suspect though that you would end up compiling as many optimized functions as there are yield points, plus one for the function entry (and possibly another for OSR). Anyway it's a bit complicated.

Also, deoptimization (bailout) becomes more difficult if you can have suspended activations, and as a technical detail, it's tricky for V8 to efficiently capture the state of an activation in a way that the garbage collector can understand.

The upshot is that tight loops in generators won't run fast. If you need speed in a loop in a generator, you will need to call out to some other function (which itself would get optimized).

**baseline**

On the other hand, I should note that generators *are* optimized for quick suspend and resume. In a generator function, V8 stores local variables on the heap instead of on the stack. You would think this would make things slow, but that is not the case. Access to locals has the same speed characteristics: both stack and heap locals are accessed by index from a register (the stack pointer or the context pointer).

There is an allocation overhead when you enter a generator to create the heap storage, but it is almost never the case that you need to copy or allocate when suspending a generator. Since the locals are on the heap already, there is nothing to do! You just write the current instruction pointer (something you know at compile-time) into a slot in the generator object and return the result.

Similarly, when resuming you have to push on a few words to rebuild the stack frame, but after that's done you just jump back to where you were and all is good.

The exception to this case is when you yield and there are temporaries on the stack. Recall in my [article on V8's baseline compiler](//wingolog.org/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler) that the full-codegen is a stack machine. It allocates slots to named locals, but temporary values go on the stack at run-time, and unfortunately V8's baseline compiler is too naive even to know what the stack height is at compile-time. So each suspension checks the stack height, and if it is nonzero, it calls out to a runtime helper to store the operands.

This run-time save would be avoidable in register machines like JavaScriptCore's baseline compiler, which avoids pushing and popping on the stack. Perhaps this note might spur the competitive urges of some fine JSC hacker, to show up V8's generators performance. I would accept JSC having faster generators if that's what it took to get generators into WebKit, at least for a while anyway :)

**abstract musings**

Many of you have read about the [Futamura projections](http://en.wikipedia.org/wiki/Partial_evaluation#Futamura_projections), in which one compiles a program via partial application of an interpreter with respect to a given input program. Though in the real world this does not appear to be a good strategy for implementing compilers, as so much of compilation is not simply about program structure but also about representation, there is a kernel of deep truth in this observation: the essence of compilation is partial evaluation of a strategy with respect to a particular program.

This observation most often helps me in the form of its converse. Whenever I find myself implementing some sort of generic evaluation strategy, there's a sign in my head that start blinking the word "interpreter". (Kinda like the "on-air" signs in studios.) That means I'm doing something that's probably not as efficient as it could be. I find myself wondering how I can make the strategy specific to a particular program.

In this case, we can think of suspending and resuming not as copying a stack frame out and in, but instead compiling specific stubs to copy out only the values that are needed, in the representations that we know that they have. Instead of looping from the bottom to the top of a frame, you emit code for each element. In this way you get a specific program, and if you manage to get here, you have a program that can be inlined too: the polymorphic "next" reduces to a monomorphic call of a particular continuation.

Anyway, [Amdahl's law](http://en.wikipedia.org/wiki/Amdahl's_law) limits the juice we can squeeze from this orange. For a tight for-of loop with a couple of optimizations that should land soon in V8, I see generator performance that is only 2 times slower that the corresponding hand-written iterators that use stateful closures or objects: 25 nanoseconds per suspend/resume versus 12 for a hand-written iterator.

25 *nanoseconds*, people. You're using jQuery and asking for data from a server in the next country. It is highly unlikely that generators will be a bottleneck in your program :)

## related articles

- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)
- [es6 generators and iteration in spidermonkey](/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey)
- [generators in v8](/archives/2013/05/08/generators-in-v8)
- [inside full-codegen, v8's baseline compiler](/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

### 4 responses

1. [Sanjoy Das](http://playingwithpointers.com) says:[11 June 2013 2:15 PM](#82b3c76dd0c3c5598ca703b6bb403ad536cb8bc9)

   \> It's trickier to do parallel compilation with OSR, as the

   \> function might have left the loop by the time recompilation

   \> finishes, but oh well.

   Another reason why parallel OSR is difficult in v8: the OSR machinery needs to be told about the pc at which the non-optimized code will jump into the optimized code (so OSR is problematic even if the pc is in the loop by the end of compilation).

2. [Ole Laursen](http://people.iola.dk/olau/) says:[11 June 2013 7:07 PM](#6e317b2faf5f6b87699f16d3c561649a838f5b0c)

   About the nanoseconds - I once wrote a little C++ iterator utility class for accessing pixels in gdk-pixbuf image data. It made it easy to process the pixels much faster than through the official interface. But it was only possible because the abstraction overhead was removed by the compiler.

3. Anonymous says:[11 June 2013 11:05 PM](#94ea38a6abcda8a617318cbf1de3a043b7ba33a9)

   I wonder how much work it would be to rework a Javascript interpreter/compiler to use a GHC-style thunk-evaluation model for lazy evaluations?

4. daishi says:[8 August 2013 3:11 PM](#5b8f5de9978d788c4053569dc803dd07ccbffeed)

   Hi, I worked on CPS transformation: https://github.com/dai-shi/continuation.js

   I was expecting generators may speed up this.

   So double overhead is a bad news.

   Is there any possibility, once the optimization is done,

   to reduce the overhead of generators less than that of hand written iterators?

Comments are closed.
