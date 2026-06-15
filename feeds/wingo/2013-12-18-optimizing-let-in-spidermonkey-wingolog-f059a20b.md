---
title: optimizing let in spidermonkey — wingolog
url: https://wingolog.org/archives/2013/12/18/optimizing-let-in-spidermonkey
published: "2013-12-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/12/18/optimizing-let-in-spidermonkey
---

# optimizing let in spidermonkey — wingolog

## [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)

18 December 2013 8:00 PM

- [javascript](/tags/javascript)
- [igalia](/tags/igalia)
- [bloomberg](/tags/bloomberg)
- [es6](/tags/es6)
- [ecmascript](/tags/ecmascript)
- [scope](/tags/scope)
- [spidermonkey](/tags/spidermonkey)
- [v8](/tags/v8)

Peoples! Firefox now optimizes `let`-bound variables!

What does this mean, you ask? Well, as you nerdy wingolog readers probably know, the new ECMAScript 6 standard is coming soon. ES6 has new facilities that make JavaScript more expressive. At the same time, though, many ES6 features stress parts of JavaScript engines that are currently ignored.

One of these areas is lexical scope. Until now, with two exceptions, all local variables in JavaScript have been "hoisted" to the function level. (The two exceptions are the exception in a catch clause, and the name of a named function expression.) Therefore JS engines have rightly focused on compiling methods at a time, falling back to slower execution strategies when lexical scopes are present.

The presence of `let` in ES6 changes this. It was a hip saying a couple years ago that " `let` is the new `var`", but in reality no one uses `let` \-\- not only because engines didn't yet implement `let` without feature flags, if they implemented it at all, but because `let` was slow. Because `let`-bound variables were on the fallback path, they were unoptimized.

But that is changing. Firefox now optimizes many kinds of `let`-bound variables, making " `let` is the new `var`" much closer to being an actual thing. V8 has made some excellent steps in similar directions as well. I'll focus on the Firefox bit here as that's what I've been working on, then go on to mention the work that the excellent V8 hackers have done.

**implementing scope in javascript**

JavaScript the language is defined in terms of a "scope chain", with respect to which local variables are looked up. Each link in the chain binds some set of variables. Lookup of a name traverses the chain from tip to tail looking for a binding.

Usually it's possible to know at compile-time where a binding will be in the chain. For example, you can know that the "x" reference in the returned function here:

```
function () {
  var x = 3;
  return function () { return x; };
}

```

will be one link up in the scope chain, and will be stored in the first named slot in that link. This is a classic implementation of lexical scope, and it plays very well with the semantics of JavaScript as specified. All engines do something like this.

Another thing to note in this case is that we don't need to store the name for "x" anywhere, as we can see through all of its uses at compile-time. In practice however, as JavaScript functions are typically compiled lazily on first call, we do store the association "x -> slot 0" in the outer function's environment.

(Could you just propagate the constant, you ask? Of course you could. But since JS compilation typically occurs one function at a time, lazily, no engine does this initially. Of course later when the optimizing compiler kicks in, it does get propagated, but that usually doesn't avoid the scope chain node allocation.)

Let's take another case.

```
function foo() { var x = 3; return x; }

```

In this case we know that the "x" reference can be found in the top link of the chain, in the first slot. Semantically speaking, that is. No JS engine implements things this way. Instead we take advantage of noticing that "x" cannot be captured by any other function. Therefore we are free to assign "x" to a slot in a stack frame, and refer to it by index directly -- without allocating a scope chain node on the heap. All engines do this.

And of course later if this function is hot, the constant exists only in a register, probably inlined to its caller. For this reason scope hasn't had a lot of work put into it. The unit of optimization -- the function -- is the same as the unit of scope. If the implementation details of scope get costly, we take advantage of run-time optimization to speculatively remove allocations, loads, and stores.

OK. What if you have some variables captured, and some not captured?

```
function foo() { var x = 3; var y = 4; return function () { return x; } }

```

Here "x" is captured, but "y" is not. Usually engines will allocate "x" on the scope chain, and "y" as a local. (JavaScriptCore is an interesting exception; it uses a strategy I haven't seen elsewhere called "lazy tear-off". Basically all variables are on the stack for the dynamic extent of the scope, whether they are captured or not. If needed, potentially lazily, an object is pushed on the scope chain that aliases the stack slots. If a scope is pushed on the scope chain, when the scope ends the current values are "torn off" the stack and copied to the heap, and the slots pointer of the scope object updated to point to the heap.)

I digress. "x" on the scope chain, "y" not. (I regress: why allocate "y" at all, isn't it dead? Yes it is dead. Optimizing compilers kill it. Baseline compilers don't bother.) So typically access to "x" goes through the scope chain, and access to "y" does not.

Calculating the set of variables that can be captured is one of the few static analyses done by JS baseline compilers. The other ones are whether we are in strict mode or not, whether nested scopes contain "with" or "eval" (thus forcing all locals to be on the scope chain), and whether the "arguments" object is used or not. There may be more, but that is the lowest common denominator of efficiency.

**implementing `let`**

`let` is part of ES6, and all browsers will eventually implement it.

Let me give you an insight as to the mindset of a browser maker. What a browser maker does when they see a feature is first to try to ignore it -- all features have a cost and not all of them pay for themselves. Next you try to do the minimum correct thing -- the thing that passes all the test suites, but imposes a minimal burden on the rest of the system. Usually this means that the feature probably works, except for the corner cases which users will file bugs for, but it is slow. Finally when there is either an internal use the browser maker cares about (Google, Mozilla to an extent) or you want to heat up a benchmark war (everyone else), you start to optimize.

The state of `let` was until recently between "ignore" and "slow". "Slow" means different things to different browsers.

The engines that have started to implement `let` are V8 (Chrome) and SpiderMonkey (Firefox). In Chrome, until recently using `let` in a function was an easy way of preventing that function from ever being optimized by Crankshaft. Good times? When writing this article I was going to claim this was still the case, but I see that in fact this is happily not true. You can Crankshaft a function that has `let` bindings. I believe, with a cursory glance, that locals that are not captured are still allocated on a scope chain -- but perhaps I am misreading things.

Firefox, on the other hand, would not Ion-compile any function with a `let`. You would think that this would not be the case, given that Firefox has had `let` for many years. If you thought this, you misunderstand how browser development works. Basically let me lay it out for you:

class struggle : marxism :: benchmarks : browsers

Get it? Benchmarks are the motor of browser history. If this bloviation has any effect, please let it be to inspire you to go and make benchmarks!

Anyway, Firefox. No Ion for `let`. The reason why you would avoid optimizing these things is clear -- breaking the "optimization unit == scope unit" equivalence has a cost. However the time has come.

What I have done is to recognize `let` scopes that have no captured locals. This class of `let` scope will now be optimized by Ion. So for example in this case:

```
function sumto(n) {
  let sum = 0;
  for (let i=0; iPreviously this would allocate a block scope around the "for".  (Incidentally, Firefox borks block scoping in this case; each iteration should produce a fresh binding.  But this is historical, will be fixed, and I digress.)  The operations that created and pushed block scopes were not optimized.  Avoiding pushing and popping scopes from the scope chain avoids this limitation, and thus we have awesome speed!The effect of simply turning on Ion cannot be denied.  In this case, the runtime of a sum up to 1e9 goes from 8.8 seconds (!) to 0.8 seconds.  Obviously it's a little micro-benchmark but that's how I'm rolling today.  The biggest optimization is to stop deoptimization.  And here's where history cranks up: V8 takes 7.7 seconds on this loop.  Gentlefolk, start your engines!At the same time, in Firefox we are still able to reify a parallel chain of "debug scopes" as needed that corresponds to the scope chain as the specification would see it.  This was the most challenging part of optimizing block scope in Firefox -- not the actual optimization, which was trivial, but optimization while retaining introspection and debuggability.future workUnhappily, scopes with let-bound variables that are captured by nested scopes ("aliased", in spidermonkey parlance) are not yet optimized.  So not only do they cause scope chain allocation, but they also don't benefit from Ion.  Boo.  Bug 942810.I should also mention that the let supported by Firefox is not the let specified in ES6.  In ES6 there is thing called a "temporal dead zone" whereby it is invalid to access the value of a let before its initialization.  This is like Scheme's "letrec restriction", and has to be enforced dynamically for similar reasons.  V8 does a great job in actually implementing this, and Firefox should do it soon.Of course, it's not clear to me how let can actually be deployed without breaking the tubes.  I think it probably can somehow but I haven't checked the latest TC39 cogitations in that regard.twisted pathsIt's been a super-strange end-of-year.  I was sure I would be optimizing SpiderMonkey generators and moving on to other things, but I just got caught up with stuff -- the for-of bits took approximately forever and then the amount of state carried in SpiderMonkey stack frames filled me with despair.  So it was that this hack, while assisting me in that goal, was not actually a planned thing.See, SpiderMonkey used to actually reserve a stack slot for the "block chain".  No, they aren't using your browser to mine for Bitcoins, though that would be a neat hack.  The "block chain" was a simulation of the spec-mandated scope chain.  But of course this might not reflect the actual implemented, optimized behavior -- but again one might want to map them to each other for debugging purposes.  It was a mess.The resulting changeset to fix this ended up so large that it took weeks to land.  Well, live and learn, right?  I remember Michael Starzinger telling me the same thing about V8 -- you just have to keep your patches small, as small as possible, and always working.  Words to the wise indeed.happy daysBut in the end we at least have some juice from this citric fruit.  This has been an Igalia joint.  Thanks very much to Mozilla's Luke Wagner for suffering through the reviews.Thanks especially to Bloomberg for making this work possible; you folks are swell.  Forward ES6!
```

## related articles

- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [arrow functions coming to chrome 45!](/archives/2015/06/18/arrow-functions-coming-to-chrome-45)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [es6 generators and iteration in spidermonkey](/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)

### 4 responses

1. anonim says:[18 December 2013 9:08 PM](#695e26a4885f7777a63d741fc6f1bae312906ded)

   So gnome-shell that uses let by default should benefit from this when porting to the next spidermonkey release right?

2. [Bob Nystrom](http://journal.stuffwithstuff.com) says:[18 December 2013 9:54 PM](#606c0d852d8e899209fb0a6397b750c4b14b19f6)

   JSC's "lazy tear-off" sounds a lot like how "upvalues" (i.e. closed-over variables) are implemented in Lua. A variable initially starts on the stack. If a function than closes over it, the function references an "upvalue", which points back to the variable on the stack. When the variable goes out of scope, the value is removed from the stack and stuffed into the upvalue which lives on the heap.

   It has the nice characteristic of playing well with Lua's single-pass compiler: since Lua doesn't know a variable will be closed over until after other code working with it has been compiled, it's critical that the variable not be moved off the stack until later.

3. Jan de Mooij says:[28 December 2013 10:55 AM](#7eddb2d3e01b4b51f6ef7426e9f0767429210ba2)

   At first glance, JSC's lazy tear-off sounds like what SpiderMonkey did before bug 659577 \[0\]. "Call objects" (objects on the scope chain) had a StackFrame pointer, and when the StackFrame was popped we'd set that pointer to null and copy the locals/arguments to the call object's slots.

   One of the problems with this was that accessing a var on the scope chain had to check if the call object had a non-null StackFrame to decide whether it should fetch the value from the StackFrame or the call object's slots.

   Another problem was that we were working on IonMonkey and, unlike the interpreter and JM, it didn't use the StackFrame layout, so we had to do something different to support functions with call objects in Ion code.

   JSC probably does something smarter than what SpiderMonkey did but having aliased vars always in the call object solved a bunch of problems for us and was a nice simplification :)

   \[0\] https://bugzilla.mozilla.org/show\_bug.cgi?id=659577

4. [Calgary Movers](http://calgarymoverspro.ca/) says:[8 February 2017 3:18 PM](#310850280ee6391752f5ed841f8d46e0895ad4de)

   This was really an interesting topic and I kinda agree with what you have mentioned here!

Comments are closed.
