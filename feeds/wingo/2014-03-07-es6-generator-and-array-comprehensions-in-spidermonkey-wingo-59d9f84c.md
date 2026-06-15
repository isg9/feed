---
title: es6 generator and array comprehensions in spidermonkey — wingolog
url: https://wingolog.org/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey
published: "2014-03-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey
---

# es6 generator and array comprehensions in spidermonkey — wingolog

## [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)

7 March 2014 9:11 PM

- [es6](/tags/es6)
- [igalia](/tags/igalia)
- [spidermonkey](/tags/spidermonkey)
- [firefox](/tags/firefox)
- [bloomberg](/tags/bloomberg)
- [compilers](/tags/compilers)
- [javascript](/tags/javascript)
- [generators](/tags/generators)

Good news, everyone: ES6 generator and array comprehensions just landed in SpiderMonkey!

Let's take a quick look at what comprehensions are, then talk about what just landed in SpiderMonkey and when you'll see it in a Firefox release. Thanks to [Bloomberg](http://www.bloomberg.com/company/) for sponsoring this work.

**[comprendes, mendes](http://www.dailymotion.com/video/xfs2w_control-machete-comprendes-mendes_music)**

Comprehensions are syntactic sugar for iteration. Unlike [for-of](//wingolog.org/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey), which processes its body for side effects, an array comprehension processes its body for its values, collecting them into a new array. Like this:

```
// Before (by hand)
var foo = (function(){
             var result = [];
             for (var x of y)
               result.push(x*x);
             return result;
           })();

// Before (assuming y has a map() method)
var foo = y.map(function(x) { return x*x });

// After
var foo = [for (x of y) x*x];

```

As you can see, array comprehensions are quite handy. They're expressions, not statements, and so their result can be passed directly to whatever code needs it. This can make your program more clear, because you aren't forced to give names to intermediate values, like `result`. At the same time, they work on any iterable, so you can use them on more kinds of data than just arrays. Because array comprehensions don't make a new closure, you can access `arguments` and `this` and even yield from within the comprehension tail.

Generator comprehensions are also nifty, but for a different reason. Let's look at an example first:

```
// Before
var bar = (function*(){ for (var x of y) yield x })();

// After
var bar = (for (x of y) x);

```

As you can see the syntactic win here isn't that much, compared to just writing out the `function*` and invoking it. The real advantage of generator comprehensions is their similarity to array comprehensions, and that often you can replace an array comprehension by a generator comprehension. That way you never need to build the complete list of values in memory -- you get laziness for free, just by swapping out those square brackets for the comforting warmth of parentheses.

Both kinds of comprehension can contain multiple levels of iteration, with embedded conditionals as you like. You can do `[for (x of y) for (z of x) if (z % 2) z + 1]` and all kinds of related niftiness. Comprehensions are almost always more concise than `map` and `filter`, with the added advantage that they are usually more efficient.

**what happened**

SpiderMonkey has had comprehensions for a while now, but only as a non-standard language extension you have to opt into. Now that comprehensions are in the draft ES6 specification, we can expose them to the web as a whole, by default.

Of course, the comprehensions that ES6 specified aren't quite the same as the ones that were in SM. The obvious difference is that SM's legacy comprehensions were written the other way around: `[x for (x of y)]` instead of the new `[for (x of y) x]`. There were also a number of other minor differences, which I'll list here for posterity:

- ES6 comprehensions create one scope per "for" node -- not one for the comprehension as a whole.

- ES6 comprehensions can have multiple "if" components, which may be followed by other "for" or "if" components.

- ES6 comprehensions should make a fresh binding on each iteration of a "for", although Firefox currently doesn't do this ( [bug 449811](https://bugzilla.mozilla.org/show_bug.cgi?id=449811)). Incidentally, for-of in Firefox has this same problem.

- ES6 comprehensions only do for-of iteration, not for-in iteration.

- ES6 generator comprehensions always need parentheses around them. (The parentheses were optional in some cases for SM's old generator comprehensions.

- ES6 generator comprehensions are ES6 generators (returning `{value, done}` objects), not legacy generators ( `StopIteration`).

I should note in particular that the [harmony wiki](http://wiki.ecmascript.org/doku.php?id=harmony:array_comprehensions) is out of date, as the feature has moved into the spec proper: [array comprehensions](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-array-comprehension), [generator comprehensions](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-generator-comprehensions).

For another fine article on ES6 generators, check out Ariya Hidayat's [piece on comprehensions](http://ariya.ofilabs.com/2013/01/es6-and-array-comprehension.html) from about a year ago.

So, ES6 comprehensions just landed in SpiderMonkey today, which means it should be part of Firefox 30, which should reach "beta" in April and become a stable release in June. You can try it out tomorrow if you use a nightly build, provided it doesn't cause some crazy breakage tonight. As of this writing, Firefox will be the first browser to ship ES6 array and generator comprehensions.

**colophon**

I had a Monday of despair: hacking at random on something that didn't solve my problem. But I had a Tuesday morning of pleasure, when I realized that my Monday's flounderings could be cooked into a delicious mid-week bisque; the hack was obvious and small and would benefit the web as a whole. (Wednesday was for polish and Thursday on another bug, and Friday on a wild parser-to-OSR-to-assembly-and-back nailbiter; but in the end all is swell.)

Thanks again to [Bloomberg](http://www.bloomberg.com/company/) for this opportunity to build out the web platform, and to Mozilla for their quality browser wares (and even better community of hackers).

This has been an [Igalia](http://www.igalia.com/) joint. Until next time!

## related articles

- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [multi-value webassembly in firefox: a binary interface](/archives/2020/04/08/multi-value-webassembly-in-firefox-a-binary-interface)
- [multi-value webassembly in firefox: from 1 to n](/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n)
- [firefox's low-latency webassembly compiler](/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler)

### 6 responses

1. [Arne Babenhauserheide](http://draketo.de) says:[7 March 2014 9:45 PM](#8a9bc905dbda178a51dddc9b2288fa6530e04ffa)

   The generators are actually nice to read - I did not expect that.

   It looks like Javascript is actually growing up to become a somewhat readable language. Thanks for your work on this!

2. [Brendan Eich](http://www.brendaneich.com/) says:[8 March 2014 8:38 AM](#6ce0253de124a6ba1fea48d63af4b4be0fe95ded)

   Hi Andy, thanks for all the great work.

   Quick note to say that the "legacy" in SpiderMonkey was all proposed for ES4, so standards track, back in the day. It was in turn based on Python 2.5 (but we avoided GeneratorExit).

   Lots of water under the bridge, but I do not believe we would have this stuff in ES6 without a bunch of us old-timers carrying water in those days.

   Maybe via some other path? Who knows. What's certain is that you did the essential ES6 implementation work, so thanks again!

   /be

3. [Brendan Eich](http://www.brendaneich.com/) says:[8 March 2014 8:39 AM](#3a20bf0a26a1125641041c3bb47fb4807b485261)

   Forgot to credit Dave Herman for turning the Pythonic RTL

   x for x in y

   around to read LTR:

   for (x of y) x

   in ES6. Good change!

   /be

4. tomás zerolo says:[8 March 2014 11:06 AM](#30da2b0da5c17e85be8edc72bd12c89aa4d9cbde)

   Hi, Andy

   thanks for your blog. Here's one tentative nitpick in return. Shouldn't

   ```
   // Before
   var bar = (function*(){ for (var x of y) yield y })();

   // After
   var bar = (for (x of y) y);

   ```

   rather be

   ```
   ... yield x ...
   ```

   respectively?

   As someone not much accustomed to Javascript syntax, this tripped me up at first.

5. Brandon says:[11 March 2014 6:05 AM](#f932f298d146a63869397dec2ce712f6c22fe4b2)

   Thanks, Andy: "comforting warmth of parenthesis" is now my phrase for the week. I enjoyed a hearty chuckle over that -- good-naturedly, I assure you.

6. [wingo](http://wingolog.org) says:[10 June 2014 8:13 AM](#fa4d1b35ff102a6b7b2d9de27cb5aa5a5f1f819e)

   Thanks for the note, tomás; fixed.

Comments are closed.
