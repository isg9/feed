---
title: generators in firefox now twenty-two times faster — wingolog
url: https://wingolog.org/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster
published: "2014-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster
---

# generators in firefox now twenty-two times faster — wingolog

## [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)

14 November 2014 8:41 AM

- [spidermonkey](/tags/spidermonkey)
- [firefox](/tags/firefox)
- [mozilla](/tags/mozilla)
- [javascript](/tags/javascript)
- [es6](/tags/es6)
- [ecmascript](/tags/ecmascript)
- [v8](/tags/v8)
- [jandem](/tags/jandem)
- [bloomberg](/tags/bloomberg)
- [igalia](/tags/igalia)

It's with great pleasure that I can announce that, thanks to Mozilla's Jan de Mooij, the new ES6 generator functions are *twenty-two times faster* in Firefox!

Some back-story, for the unawares. There's a new version of JavaScript coming, ECMAScript 6 (ES6). Among the new features that ES6 brings are [generator functions: functions that can suspend](//wingolog.org/archives/2013/05/08/generators-in-v8). Firefox's JavaScript engine, [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey), has had [support for generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators) for many years, long before other engines. This support was [upgraded to the new ES6 standard last year](//wingolog.org/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey), thanks to sponsorship from [Bloomberg](http://bloomberg.com), and was shipped out to users in [Firefox 26](https://developer.mozilla.org/en-US/Firefox/Releases/26#JavaScript).

The generators implementation in Firefox 26 was quite basic. As you probably know, [modern JavaScript implementations have a number of tiered engines](http://wingolog.org/archives/2011/07/05/v8-a-tale-of-two-compilers). In the case of SpiderMonkey there are three tiers: the interpreter, the baseline compiler, and the optimizing compiler. Code begins execution in the interpreter, which is the quickest engine to start. If a piece of code is hot -- meaning that lots of time is being spent there -- then it will "tier up" to the next level, where it is analyzed, possibly optimized, and then compiled to machine code.

Unfortunately, [generators in SpiderMonkey have always been stuck at the lowest tier, the interpreter](https://bugzilla.mozilla.org/show_bug.cgi?id=903457). This is because of SpiderMonkey's choice of implementation strategy for generators. Generators were implemented as "floating interpreter stack frames": heap-allocated objects whose shape was exactly the same as a stack frame in the interpreter. This had the advantage of being fairly cheap to implement in the beginning, but ultimately it made them unable to take advantage of JIT compilation, as JIT code runs on its own stack which has a different layout. The previous strategy also relied on trampolining through a helper written in C++ to resume generators, which killed optimization opportunities.

The solution was to represent suspended generator objects as snapshots of the state of a stack frame, instead of as stack frames themselves. In order for this to be efficient, last year we did a number of [block scope optimizations](http://wingolog.org/archives/2013/12/18/optimizing-let-in-spidermonkey) to try and reduce the amount of state that a generator frame would have to restore. Finally, around March of this year we were at the point where we could refactor the interpreter to implement generators on the normal interpreter stack, with normal interpreter bytecodes, with the vision of being able to JIT-compile those bytecodes.

I ran out of time before I could land [that patchset](https://bugzilla.mozilla.org/show_bug.cgi?id=987560); although the patches were where we wanted to go, they actually caused generators to be even slower and so they languished in Bugzilla for a few more months. Sad monkey. It was with delight, then, that a month or so ago I saw that SpiderMonkey JIT maintainer Jan de Mooij was interested in picking up the patches. Since then he has been hacking off and on at getting my old patches into shape, and ended up applying them all.

He went further, [optimizing stack frames to not reserve space for "aliased" locals (locals allocated on the scope chain)](https://bugzilla.mozilla.org/show_bug.cgi?id=1090491), [speeding up object literal creation in the baseline compiler](https://bugzilla.mozilla.org/show_bug.cgi?id=1097890) and finally has implemented [baseline JIT compilation for generators](https://bugzilla.mozilla.org/show_bug.cgi?id=1093573).

So, after all of that perf nargery, what's the upshot? Twenty-two times faster! In this microbenchmark:

```
function *g(n) {
    for (var i=0; iBefore, it took SpiderMonkey 980 milliseconds to complete on Jan's machine.  After?  Only 43!  It's actually marginally faster than V8 at this point, which has (temporarily, I think) regressed to 45 milliseconds on this test.  Anyway.  Competition is great and as a committer to both projects I find it very satisfactory to have good implementations on both sides.As in V8, in SpiderMonkey generators cannot yet reach the highest tier of optimization.  I'm somewhat skeptical that it's necessary, too, as you expect generators to suspend fairly frequently.  That said, a yield point in a generator is, from the perspective of the optimizing compiler, not much different from a call site, in that it causes all locals to be saved.  The difference is that locals may have unboxed representations, so we would have to box those values when saving the generator state, and unbox on restore.Thanks to Bloomberg for funding the initial work, and big, big thanks to Mozilla's Jan de Mooij for picking up where we left off.  Happy hacking with generators!
```

## related articles

- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [arrow functions coming to chrome 45!](/archives/2015/06/18/arrow-functions-coming-to-chrome-45)

### 7 responses

1. Matthias says:[14 November 2014 3:53 PM](#59ccd1aeac6262b09c04f072e464a088ce767a58)

   This count for which Version(s) please ?

2. Tom Schuster says:[14 November 2014 4:33 PM](#940f5d4e59a09984a1b218637644282c6ca6f5ff)

   Wingo, thank you for taking the time to write this blog post!

   @Mathias If you look at https://bugzilla.mozilla.org/show\_bug.cgi?id=1093573 you will see a "Target Milestone" of mozilla36, which is Firefox 36, or at this time Firefox nightly.

3. [Arne Babenhauserheide](http://draketo.de) says:[14 November 2014 5:11 PM](#3ee6b15a59b13aafbb5845adacc7bc5a0454f694)

   Congratulations! This is awesome!

   And it’s great to see that your work paid off!

   For the speed, I got this link: http://arewefastyet.com/

4. paul says:[14 November 2014 6:02 PM](#e047c04961068be95d8e09f5619e15693aa06ee4)

   Nice! Will there be real coroutines at some point? That would allow getting rid of the horrid callback nonsense in node.js.

5. [db48x](http://db48x.net/) says:[14 November 2014 7:39 PM](#1644aa179a9acd5e8cbbb70caca13ed68ef6fff1)

   Paul: Yes, you can use yield to eliminate callbacks. Rather than explain, I'll just show an example: http://db48x.net/yield/yield.js

6. ElectricPrism says:[14 November 2014 7:54 PM](#2138b687486463c88cb04d5a42392341c8ed0ba9)

   Had to say, kickass work bro.

7. William ML Leslie says:[17 November 2014 7:31 AM](#f6160ee449048511a0c334f1a9fe512c82b770fd)

   Hey Andy,

   Just wanted to let you know that your less-than sign in the code block seems not to be escaped in the RSS feed.

Comments are closed.
