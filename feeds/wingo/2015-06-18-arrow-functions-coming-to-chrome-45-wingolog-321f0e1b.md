---
title: arrow functions coming to chrome 45! — wingolog
url: https://wingolog.org/archives/2015/06/18/arrow-functions-coming-to-chrome-45
published: "2015-06-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/06/18/arrow-functions-coming-to-chrome-45
---

# arrow functions coming to chrome 45! — wingolog

## [arrow functions coming to chrome 45!](/archives/2015/06/18/arrow-functions-coming-to-chrome-45)

18 June 2015 4:41 PM

- [v8](/tags/v8)
- [chrome](/tags/chrome)
- [javascript](/tags/javascript)
- [es6](/tags/es6)
- [chromium](/tags/chromium)
- [igalia](/tags/igalia)
- [bloomberg](/tags/bloomberg)

It's been a long time coming, but I just flipped the bit in V8 that will ship arrow functions in Chrome 45! Woo hoo!

You probably know, but arrow functions are a new way to write functions in JavaScript. They look like this:

```
// Two arguments, body implicitly returned.
(x, y) => x + y

// With just one argument, no parentheses needed.
x => x * 2

// Body can have braces too; in that case use "return".
x => { return x * 2 }

```

Relative to the other kind of function that is written like `function (x) { return x * 2 }`, arrow functions don't define `this` or `arguments` in their bodies, instead capturing these values from the environment. There are a couple of other minor differences, too, but instead of writing about them here I'll just point to the great [article by Jason Orendorff of the SpiderMonkey team](https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/).

Arrow functions are part of the JavaScript language standard that was called "ECMAScript 6" or ES6, and I guess you could still call it that. It seems like a silly thing for the committee to do to throw away all their branding like that but they decided to rename it [ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/), which I'm sure is a link that the pedants are glad I have included. The upshot is that the standard is now final, gold master, etched in stone, which from an implementor's perspective is a relief. You can practically feel the anxiety ebbing away by the happy rate at which commits bubble out of source repositories and into shipping browsers, free from the fear that some spec change will force the hack-stream to change course.

From the V8 side, our arrow function implementation has also been a long time coming. My colleague Adrián Pérez did the first half of the work, and I picked up on the back end of things. It seems like such a small feature and in many ways it is, but still it took a long time. Now I know that my readers are a bunch of nerds and many of you like implementing languages, so you might appreciate these nargish points.

One of the first bits is that arrow functions are hard to parse. Consider, this is a valid JavaScript expression:

```
(x,y)

```

It's a "comma expression" that will evaluate `x` then `y` and its result will be the result of evaluating `y`. But add an arrow on after the end and you get not an expression but a formal parameter list:

```
(x,y)=>x+y

```

Now you might think, well OK, when you see an arrow, rewind the input stream and parse in "arrow function mode". Indeed that would be fine, but not in combination with some additional ES6 features, optional and destructuring arguments. Optional arguments look like this:

```
(x=42)=>x

```

The `=42` part is the expression that will be evaluated to give x a value, if the function is called with no arguments. Note that this bit is still under implementation in V8 so you can't try it in your browser. An optional argument initializer is an expression and not a value, so you can also have:

```
(x=(x)=>42)=>x

```

Combined, this makes rewinding the token stream a proposition of exponential complexity, which is a no-go for a production JavaScript parser. Parsers are on the hot path for page-load times and no browser vendor wants to introduce a pathological case into their page load.

Instead, V8 does something I hadn't seen before. It keeps an open mind about whether something is a comma expression or a formal parameter list of an arrow function, and only makes a decision when it sees the `=>` (or not). As it parses, V8 records places that it *would* signal an error for either a parameter list or for an expression, and then when that superimposed wave function collapses it checks that the production is valid, signalling the appropriate error if not. I thought this was a really neat trick, so if you're into that thing see [expression classifier](https://chromium.googlesource.com/v8/v8/+/541b6c39e0ecae1c070f51fae8e9e3dab18d278c/src/expression-classifier.h) to see those details.

The other thing that's tricky about arrow functions is the `this` binding. In JavaScript, `this` is basically a hidden parameter passed to a function when it is called. Calling a function like `o.f()` passes the value of `o` to `f` as its `this` parameter. If instead `f()` is called directly, like with no dot before the call, then `undefined` is passed as `this`. Also for sloppy-mode functions, if the passed `this` value isn't an object, then the global object instead is assigned to `this`. Finally outside a function, `this` is bound to the global object.

OK, I know all of you know these things. Thing is, you always have a `this`, and although it's like a variable it's not a valid variable name, and before ES6 nothing could capture its value, because each function has its own `this` value. Perhaps you see where I'm going with this (ahem) now. Arrow functions introduce a function scope that doesn't have a `this` value, and that indeed might capture some other scope's `this` value, forcing it to be context-allocated. Other parts of ES6 can actually force assignment to `this`, like a `super` call, and that assignment can actually come from within an arrow function. Zounds! A simple concept, but there was a lot of incidental complexity in V8 around the implementation. Between Adrián and myself it took like three months to fix `this` usage in V8 to always just go through the (possibly context-allocated) variable, and there are still probably some devtools bugs to find in the upcoming weeks.

Performance-wise, arrow functions are just like functions. They should be just as fast as if you wrote them with `function`. So use them with joy, use them with abandon, use them judiciously -- however you decide you use them, don't let perf influence your decision one way or the other.

That's about it! Like all of my JS engine work over the past couple years, this hacking was sponsored by fabulous folks over at [Bloomberg](http://www.bloomberg.com/), so big ups to them. From me and Adrián at [Igalia](http://igalia.com/), until next time! We leave you to puzzle out what this bit of JavaScript evaluates to:

```
(({},{},({},{})=>({},{}))=>(({},{})=>({},{}),{},{}))({},{})

```

Happy hacking!

## related articles

- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)

### 5 responses

1. ssube says:[18 June 2015 8:53 PM](#fc0c14c442cacaa6f57a555fd653478d8a391391)

   The example given at the end of the post is rejected by both Babel and Traceur with syntax errors, but looks like it should be a function that returns an empty object.

2. [Awal Garg](http://awal.js.org) says:[18 June 2015 8:55 PM](#0daa35dbedcdc245857c534f90f2caa187e5e838)

   Great post.

   \> We leave you to puzzle out what this bit of JavaScript evaluates > to:

   \> (({},{},({},{})=>({},{}))=>(({},{})=>({},{}),{},{}))({},{})

   I am not sure if this is intended but it appears to be invalid code. Is the answer a \`Syntax error\`? If not, I am not sure how one can declare a function as a parameter without a name (since a function is not a valid ES6 destructuring pattern either). Leaving out the \`arguments\` and \`this\` behavior, this is what the equivalent es5 code will be:

   \`\`\`javascript

   function ({}, {}, function ({}, {}) { return {}, {} }) {

   return function ({}, {}) { return ({}, {}); }, {}, {};

   }({},{});

   \`\`\`

   Removing the part causing the syntax error (the function in the parameter declaration), it becomes:

   \`\`\`javascript

   function ({}, {}) {

   return function ({}, {}) { return ({}, {}); }, {}, {};

   }({},{});

   \`\`\`

   Which will return a function which returns \`{}\`. Any insights?

   Thanks!

3. [Jeff Walden](http://whereswalden.com/) says:[19 June 2015 9:20 PM](#d12f6d069b222d65dfad681532efee681e1a49ef)

   Concur with other commenters: `({},{},({},{})=>({},{}))` can only be a parenthesized comma-separated list of expressions, because the arrow function isn't a valid parameter (and, I think, wouldn't be starting with that opening `(` on the argument list). As to what this *should* be, I haven't the foggiest. :-) Comes from being too close to all this nonsense, I see too much sense in it.

   *"Help, I'm trapped on an ECMAScript implementation team"!*

4. [Aristotle Pagaltzis](http://plasmasturm.org/) says:[22 June 2015 5:25 AM](#a87a6e3e9a43cae5e6c31cbf1d889d876a7e6cc3)

   > Instead, V8 does something I hadn't seen before.

   You might be interested in [Marpa](https://jeffreykegler.github.io/Marpa-web-site/), a parsing algorithm that tracks all possible parses at all times – among other related properties, which together allow it to easily parse grammars that would traditionally be considered ambiguous, or to cleanly and easily parse elliptical inputs by inventing tokens on the fly as needed (based on what set of possible tokens it expects to see next). E.g. it can do parsing for BNF grammars that are both left- and right-recursive, and it’s linear for almost all grammars it can parse.

5. Andy says:[23 June 2015 5:20 AM](#b0f97bf95f922ec615c023fa0694c7251f11588d)

   You mention that using arrow functions won't affect performance. Does that mean that v8 can optimize functions that use them? All the other ES6 features I've tried still seem to permanently deopt the containing function.

Comments are closed.
