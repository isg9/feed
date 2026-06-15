---
title: generators in v8 — wingolog
url: https://wingolog.org/archives/2013/05/08/generators-in-v8
published: "2013-05-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/05/08/generators-in-v8
---

# generators in v8 — wingolog

## [generators in v8](/archives/2013/05/08/generators-in-v8)

8 May 2013 7:31 PM

- [v8](/tags/v8)
- [ecmascript](/tags/ecmascript)
- [javascript](/tags/javascript)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [delimited continuations](/tags/delimited%20continuations)
- [continuations](/tags/continuations)
- [promises](/tags/promises)

Hey y'all, ES6 generators have landed in V8! Excellent!

Many of you know what that means already, but for those of you that don't, a little story.

A few months ago I was talking with Andrew Paprocki over at [Bloomberg](http://bloomberg.com/). They use JavaScript in all kinds of ways over there, and as is usually the case in JS, it often involves communicating with remote machines. This happens on the server side and on the client side. JavaScript has some great implementations, but as a language it doesn't make asynchronous communication particularly easy. Sure, you can do anything you want with node.js, but it's pretty easy to get stuck in callback hell waiting for data from the other side.

Of course if you do a lot of JS you learn ways to deal with it. The eponymous [Callback Hell](http://callbackhell.com/) site lists some, and lately many people think of [Promises](http://ianbishop.github.io/blog/2013/01/13/escape-from-callback-hell/) as the answer. But it would be nice if sometimes you could write a function and have it just suspend in the middle, wait for your request to the remote computer to come through, then continue on.

So Andrew asked if me if we could somehow make asynchronous programming easier to write and read, maybe by implementing something like C#'s [`await`](http://msdn.microsoft.com/en-us/library/vstudio/hh156528.aspx) operator in the V8 JavaScript engine. I told him that the way into V8 was through standards, but that fortunately the upcoming ECMAScript 6 standard was likely to include an equivalent to `await`: generators.

**ES6 generators**

Instead of returning a value, when you first call a generator, it returns an iterator:

```
// notice: function* instead of function
function* values() {
  for (var i = 0; i < arguments.length; i++) {
    yield arguments[i];
  }
}

var o = values(1, 2, 3);  // => [object Generator]

```

Calling `next` on the iterator resumes the generator, lets it run until the next `yield` or `return`, and then suspends it again, resulting in a value:

```
o.next(); // => { value: 1, done: false }
o.next(); // => { value: 2, done: false }
o.next(); // => { value: 3, done: false }
o.next(); // => { value: undefined, done: true }

```

Maybe you're the kind of person that likes imprecise, incomplete, overly abstract analogies. Yes? Well generators are like functions, with their bodies taken to the first derivative. Calling `next` integrates between two yield points. Chew on that truthy nugget!

**asynchrony**

Anyway! Supending execution, waiting for something to happen, then picking up where you left off: put these together and you have a nice facility for asynchronous programming. And happily, it works really well with promises, a tool for asynchrony that lots of JS programmers are using these days.

[Q](https://github.com/kriskowal/q) is a popular promises library for JS. There are some [250+](https://npmjs.org/package/q) packages that depend on it in NPM, node's package manager. So cool, let's take their example of the "Pyramid of Doom" from the github page:

```
step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // Do something with value4
            });
        });
    });
});

```

The promises solution does at least fix the pyramid problem:

```
Q.fcall(step1)
.then(step2)
.then(step3)
.then(step4)
.then(function (value4) {
    // Do something with value4
}, function (error) {
    // Handle any error from step1 through step4
})
.done();

```

But to my ignorant eye, some kind of solution involving a straight-line function would be even better. Remember what generators do: they suspend computation, wait for someone to pass back in a result, and then continue on. So whenever you would register a callback, whenever you would chain a `then` onto your promise, instead you suspend computation by yielding.

```
Q.async(function* () {
  try {
    var value1 = yield step1();
    var value2 = yield step2(value1);
    var value3 = yield step3(value2);
    var value4 = yield step4(value3);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
});

```

And for a super-mega-bonus, we actually get to use try and catch to handle exceptions, just as Gods and Brendan intended.

Now I know you're a keen reader and there are two things that you probably noticed here. One is, where are the promises, and where are the callbacks? Who's making these promises anyway? Well you probably saw already in the second example that using promises well means that you have functions that return promises instead of functions that take callbacks. The form of functions like `step1` and such are different when you use promises. So in a way the comparison between the pyramid and the promise isn't quite fair because the functions aren't quite the same. But we're all reasonable people so we'll deal with it.

Note that it's not actually necessary that any of the `stepN` functions return promises. The promises library will lift a value to a promise if needed.

The second and bigger question would be, how does the generator resume? Of course you've already seen the answer: the whole generator function is decorated by [`Q.async`](https://github.com/kriskowal/q/tree/master/examples/async-generators), which takes care of resuming the generator when the yielded promises are fulfilled.

You don't have to use generators of course, and using them with Q does mean you have to understand more things: not only standard JavaScript, but newfangled features, and promises to boot. Well, if it's not your thing, that's cool. But it seems to me that the widespread appreciation of `await` in C# bodes well for generators in JS.

**ES6, unevenly distributed**

`Q.async` has been part of Q for a while now, because Firefox has been shipping generators for some time.

Note however that the current ES6 draft specification for generators is slightly different from what Firefox ships: calling `next` on the iterator returns an object with `value` and `done` properties, whereas the SpiderMonkey JS engine used by Firefox uses an exception to indicate that the iterator is finished.

This change was the result of some [discussions at the TC39 meeting in March](https://github.com/rwldrn/tc39-notes/blob/master/es6/2013-03/mar-12.md#412-stopiterationgenerator), and hasn't made it into a draft specification or to the [harmony:generators](http://wiki.ecmascript.org/doku.php?id=harmony:generators) page yet. All could change, but there seems to be a consensus.

I made a patch to Q to allow it to work both with the old SpiderMonkey-style generators as well as with the new ES6 style, and something like it should go in soon.

**give it to me already!**

So yeah, generators in V8! I've been working closely with V8 hackers Michael Starzinger and Andreas Rossberg on the design and implementation of generators in V8, and I'm happy to say that it's all upstream now, modulo `yield*` which should go in soon. Along with other future ES6 features, it's all behind a runtime flag. Pass `--harmony` or `--harmony-generators` to your V8 to enable it.

Barring unforeseen issues, this will probably see the light in [Chromium 29](http://www.chromium.org/developers/calendar), corresponding to V8 3.19. For now though this hack is so hot out of the fire that it hasn't even had time to cool down and make it to the Chrome Canary channel yet. Perhaps within a few weeks; whenever the V8 dependency in Chrome gets updated to the 3.19 tree.

As far as Node goes, they usually track the latest stable V8 release, and so for them it will probably also be a few weeks before it goes in. You'll still have to run Node with the `--harmony` flag. However if you want a sneak preview of the future, I uploaded a [branch of Node with V8 3.19](https://github.com/andywingo/node/tree/v8-3.19) that you can build from source. It might mulch your cat, but life is not without risk on the `bleeding_edge`.

Finally for Q, as I said ES6 compatibility should come soon; track the progress or check out your own copy [here](https://github.com/kriskowal/q/pull/288).

**final notes**

Thanks to the V8 team for their support in this endeavor, especially to Michael Starzinger for enduring my constant questions. There's lots of interesting things in the V8 pipeline coming out of Munich. Thanks also to Andrew Paprocki and the [Bloomberg](http://bloomberg.com/) crew for giving me the opportunity to fill out this missing piece of real-world ES6. The Bloomberg folks really get the web.

This has been a hack from [Igalia](http://www.igalia.com/), your friendly neighborhood browser experts. Have fun with generators, and happy hacking!

## related articles

- [inside full-codegen, v8's baseline compiler](/archives/2013/04/18/inside-full-codegen-v8s-baseline-compiler)
- [on generators](/archives/2013/02/25/on-generators)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)

### 12 responses

1. [C. Scott Ananian](http://cscott.net) says:[8 May 2013 8:42 PM](#369d00a0ff7ee2a237382e649f452798d944f2c8)

   We've been using yield in spidermonkey to do this for a while:

   http://blog.ometer.com/2010/11/28/a-sequential-actor-like-api-for-server-side-javascript/

   node-fibers also has some decent node-integration ideas; it might be worth stealing stuff from there.

2. [Dave Herman](http://calculist.org) says:[8 May 2013 9:22 PM](#e31025ddde16f05f9a6e4a9b4a574540a25ae3e1)

   Congrats, and thank you for helping to spread ES6 functionality to more JavaScript engines! You made the web better today. <3

   You might want to take a look at http://taskjs.org, which I created a few years ago for the same purposes you talk about here. I do have a few changes I need to make, since it was built for Firefox's generators implementation, which need to be updated to the latest version of the ES6 design. And I want to eliminate the (non-Promises/A+-compliant!) promises implementation from it and reuse an existing promise library like RSVP.js. Once I make these changes it should work nicely with V8's generators implementation.

   Dave

3. [James Long](http://jlongster.com) says:[9 May 2013 3:19 PM](#d027e1c8fdf903edbf3e4f299c891843bfe3f554)

   This is amazing, great work! I can't imagine this \*not\* rocking the node world. If we can get the cooperative threading libraries to stabilize, it seems like we won't have to talk about promises so much anymore.

4. Matt Mastracci says:[9 May 2013 7:06 PM](#7fe806ee2d3909a26f0aec4856f34d41fdae5b5c)

   Should Function.toString() be returning function\* instead of function?

   \> function\* foo() { }

   undefined

   \> foo.toString()

   'function foo() { }'

5. Matt Mastracci says:[9 May 2013 7:15 PM](#d22ba6eb53abe0d248de19c68ad55d18e457e358)

   One more bug - calling the constructor of a generator function crashes node:

   \> function\* foo() { }

   undefined

   \> foo.constructor()

   Illegal instruction: 4

6. [Andy WIngo](http://wingolog.org/) says:[10 May 2013 6:05 AM](#639656d79325f14fe72790df1d67f683141f4d3f)

   Hi Matt, are you sure you are on the v8-3.19 branch in my repository? Generators should definitely print with function\* there. It's a little confusing as 3.18 also has the first half of the generators work, so it's easy to think you're on the right branch when in reality not everything is working.

   The constructor() bug is a real one though. Thanks for the report. If you have any more, send them to me at wingo@igalia.com -- and after a couple weeks or so, the v8 bug tracker is probably the right place.

7. Brandon says:[10 May 2013 10:52 PM](#e707bb142f0a6901bac22c35bddd14f9ecdc7f27)

   Very cool. I can see this being quite nice for JavaScript UI, which is also asynchronous, and which also tends to degenerate into a maze of callbacks invoking callbacks (which queue timeouts that invoke callbacks that queue timeouts.. ). Pyramid of doom indeed. For example, say I am drawing to a canvas. It might be convenient to implement animations as generators that yield once per frame. You step an animation simply by resuming the generator, and the animation is done when the generator exits. You can write your animations in whatever style feels natural, so long as they yield once per frame.

   Are these generators like Lua co-routines, or like Python generators? What happens when foo calls bar, and bar yields?

8. [Michael Hart](http://github.com/mhart) says:[14 May 2013 1:59 AM](#b53baa0c7280165f58afddded10e1de560a07ced)

   v8 3.19 has been included in node.js v0.11.2: http://blog.nodejs.org/2013/05/13/node-v0-11-2-unstable/

   Just tested out your basic generator example (with node --harmony) and it works a treat!

9. Arpad Borsos says:[15 May 2013 4:33 PM](#4a234a3fa5ecaa60add5449dbe740217f17d4db1)

   So TC39 agreed to drop \`StopIteration\`? Thank god! Exceptions should not be used for regular control flow, otherwise we would land in java land, like this: https://gist.github.com/Swatinem/5585263

10. [Staffan Eketorp](http://staeke.wordpress.com) says:[2 September 2013 6:45 AM](#de7ca85794f745ea1e986048bb0240a3d5c37d75)

    Hey. Thanks for a nice blog post. I know I'm a little late, but it inspired me to look out for some libraries and experiment with new builds of canary and node. Since I believed there were a few things missing in Q and Galaxy I wrote my own library for generator support, Y-yield. I'd be super happy if you were able to give it some feedback or mention it in some blog/tweet.

    Thanks

11. Staffan Eketorp says:[2 September 2013 6:55 AM](#021501b2ba69d8c77e4d637718069c8526a8c52d)

    By the way - I forgot the link: https://npmjs.org/package/yyield

12. Vedran Rodic says:[23 April 2014 2:42 PM](#a328a08a07bc4b27b655871516a898355ce5654c)

    In nodeup podcast episode 46 you mentioned a certain bug in v8 when yield waits for too long. I guess this has been fixed by now?

    I'm sorry, I know you mentioned in the podcast that we should not ask you about this one (presuming because you know about it already), but I just couldn't find the bug number to confirm it has been solved.

Comments are closed.
