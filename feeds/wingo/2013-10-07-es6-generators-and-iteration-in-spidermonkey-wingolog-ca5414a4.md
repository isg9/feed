---
title: es6 generators and iteration in spidermonkey — wingolog
url: https://wingolog.org/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey
published: "2013-10-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey
---

# es6 generators and iteration in spidermonkey — wingolog

## [es6 generators and iteration in spidermonkey](/archives/2013/10/07/es6-generators-and-iteration-in-spidermonkey)

7 October 2013 3:17 PM

- [es6](/tags/es6)
- [ecmascript](/tags/ecmascript)
- [js](/tags/js)
- [spidermonkey](/tags/spidermonkey)
- [v8](/tags/v8)
- [igalia](/tags/igalia)
- [generators](/tags/generators)
- [iteration](/tags/iteration)
- [coroutines](/tags/coroutines)

It is my great pleasure to announce that SpiderMonkey, the JavaScript engine of the Firefox browser, just got support for ES6 generators and iteration!

**the chase**

As I have written before, [generators are functions that can suspend](//wingolog.org/archives/2013/05/08/generators-in-v8). SpiderMonkey has had generators since Firefox 2 (really!), but the new thing is that now their generators are *standard*, and *available by default*. No need to futz with Firefox-specific "language versions": there is just [1JS](https://mail.mozilla.org/pipermail/es-discuss/2011-December/019112.html) now.

My last article extolled the virtues of generators for asynchronous programs, and since then there have been a number of great articles and talks around the web on that topic. Generators are a kind of coroutine, and coroutines work well for asynchronous computation.

But that's not all that coroutines are good for. Another great application of generators is in [iteration](//wingolog.org/archives/2013/02/25/on-generators), where you have a producer that knows how to produce values, and a consumer that knows how to process them. Being able to write the producer as a coroutine is great win.

**words, words, words**

This is a bit abstract, so let me show some code. Let's say you are implementing an auto-complete widget, so you use a [trie](http://en.wikipedia.org/wiki/Trie). Here is a simple trie:

```
function Trie() {
  this.kids = Object.create(null);
  this.terminal = false;
}

Trie.prototype.insert = function (str) {
  if (str.length == 0) {
    this.terminal = true;
  } else {
    var node = this.kids[str[0]];
    if (!node)
      this.kids[str[0]] = node = new Trie;
    node.insert(str.slice(1));
  }
}

```

This kind of data structure is nice because it encodes the minimal amount of information. Unlike some of the examples on Wikipedia, the information from the prefix is not contained in the suffix.

Let's add some words:

```
var db = new Trie;
['fee', 'fie', 'fo', 'foo'].forEach(db.insert.bind(db));

```

Now, how do we get the completions?

```
Trie.prototype.completions = function (prefix) {
  // ?
}

```

It would be nice to avoid making an array of all of the possible completions, because who knows how many we're actually going to show. One of the whole points of using a trie was to avoid storing all of the strings into memory.

Generators offer a nice solution to this problem:

```
Trie.prototype.strings = function* () {
  if (this.terminal) yield '';
  for (var k in this.kids)
    for (var suffix of this.kids[k].completions(''))
      yield k + suffix;
}

Trie.prototype.completions = function* (prefix) {
  if (prefix.length) {
    var node = this.kids[prefix[0]];
    if (node !== undefined)
      for (var suffix of node.completions(prefix.slice(1)))
        yield prefix[0] + suffix;
  } else {
    yield* this.strings();
  }
}

```

Here you see that the information that is part of the prefix is now present implicitly, as part of the control stack. Operations on recursive data structures are naturally expressed with recursive functions.

Note that if I call `db.strings()`, nothing happens -- not even if there are millions of strings in the trie. Values get produced by the coroutine only as the consumer asks for them.

Speaking of consumers, "for-of" is usually the consumer that you want, as it is the built-in ES6 language construct for iteration. For example:

```
js> for (var word of db.completions('fo')) print(word)
fo
foo
js> for (var word of db.completions('f')) print(word)
fee
fie
fo
foo
js> for (var word of db.completions('q')) print(word)
js>

```

This code is not just part of some future standard: it's available now in Firefox nightlies, out of the box. It's also available in Chrome, if you enable "experimental features".

**niceties**

What is also in Firefox, but not yet in Chrome, is support for custom iterators. Wouldn't it be nice if you could just say:

```
for (var word of db)
  print(word);

```

Well you can! But here it gets complicated.

The right-hand side of the for-of statement, in this case `db`, is the *iterable*. The *iterator* is derived from the iterable by calling the iterable's special `@@iterator` method. The `@@` indicates that this method is keyed by a "well-known symbol".

Symbols are a new kind of data type in ES6, and their salient characteristic is that they can be used as unique property keys. You can imagine that in the future there is some piece of the JavaScript engine that, on startup, does something like this:

```
function Symbol() {}
Symbol.iterator = new Symbol;

```

and then uses that `Symbol.iterator` value as a property key.

However, Firefox does not implement symbols yet. In the meantime Firefox uses a special string as a stand-in for `@@iterator`. This name is not publicised anywhere, but you can get the value of `@@iterator` using this future-proof code:

```
const iterator = (function() {
  try {
    for (var _ of Proxy.create({get: function(_, name) { throw name; } }))
      break;
  } catch (name) {
    return name;
  }
  throw 'wat';
})();

```

However2! V8 implements for-of but not the `@@iterator` protocol, so if you target V8 also, you might find [this gist](https://gist.github.com/andywingo/6712480) to be useful.

Given `iterator`, we can now make `Trie` iterable directly:

```
Trie.prototype[iterator] = Trie.prototype.strings;

js> for (var word of db)
      print(word);
fee
fie
fo
foo

```

**how did we get here?**

That's the status: Firefox does ES6 generators and iterators. Excellent!

So, I have to toot my toot some bells and ring some horns here. I implemented ES6 generators and iteration in SpiderMonkey as part of my work with [Igalia](http://igalia.com/), your friendly neighborhood browser consultancy. I did the same in V8 a few months ago.

It seems odd, on the face of it, to hack on core components of two major browser engines, while not being employed by either associated company. We think of Google, Mozilla, Apple, and Microsoft as the ones that are really "driving the web", either because it's strategic for them, or because they are shamed by their competitors into doing so. But that's not the totality of the browser ecosystem.

To develop the web platform, there is a simple condition: you have to ship a browser, or components of a browser (e.g. a JS engine). If you do that, you can usefully invest in improving your tools, to make them better for you in some way. The magical part is that when you improve a browser, you improve the web.

Since three of the four main browser engines are Free Software, it's possible for third parties that ship browsers to meaningfully contribute to them -- and happily, this improves not only their products but also the web as a whole.

So for ES6 generators and iteration, big ups to [Bloomberg](http://bloomberg.com), who sponsored this work. More expressive language features helps them better develop their products, both on the server and client side. Because they deploy V8 and SpiderMonkey components, they were in this magical position of being able to usefully invest in the web platform, making it better for all of us. Thanks!

Thanks also to the great Mozilla folks, who were very kind, helpful, and forgiving of my mozilla-rookie mistakes. It was also a privilege to be able to get such a close insight into the workings of Firefox development. Thanks especially to Jason Orendorff and Jeff Walden for their detailed patch reviews, and to Till Schneidereit for enduring many stupid questions over IRC :) Y'all are great!

**next**

I have a richly variegated set of minor bugs to fix up with this work, so while I've still got this all paged in, do be sure to give a try to the Firefox nightlies and see if this works for you. File bugs in [Bugzilla](https://bugzilla.mozilla.org/) and copy me ( `:wingo`) on them. Thanks in advance for submitting bugs there and not in the blog comments :) Happy hacking!

## related articles

- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [ecmascript generators from a performance perspective](/archives/2013/06/11/ecmascript-generators-from-a-performance-perspective)
- [on generators](/archives/2013/02/25/on-generators)

### 8 responses

1. [John Cowan](http://www.ccil.org/~cowan) says:[9 October 2013 0:18 AM](#0ee49e1a50b63ff61bd8e61d4b830fb93e14a503)

   There are two kinds of coroutines: Python style, where you can only yield from the top-level procedure; and Lua style, where you can yield from any dynamically nested procedure. Which kind are these?

2. [wingo](http://wingolog.org/) says:[9 October 2013 9:13 AM](#eec224bed7fb98584a0a7b70c6d217315181d765)

   These are shallow coroutines; see [David Herman's article](http://calculist.org/blog/2011/12/14/why-coroutines-wont-work-on-the-web/) for a discussion. They are not as expressive as deep coroutines, but this is mitigated by yield\*, which allows one shallow coroutine to delegate to another.

3. al\_shopov says:[9 October 2013 11:22 AM](#172a75da01b04793aaf372d4520eed0ae4721fc1)

   John,

   As far as I read it and experiment - you can yield from any function/method - no matter how deeply nested.

   I am not sure what you mean by "any dynamically nested procedure". Do you mean that the yielded values and the state of the different iteration functions are somehow connected? I would believe you can do this with a common closure that the inner procedures all share.

   This changing of state will be the common JS change of the variables in the closure, otherwise the execution state of each individual iterator/generator (the point to the last yield they already reached) will stay the same.

   Perhaps Andy will chime in about this.

4. [John Cowan](http://www.ccil.org/~cowan) says:[9 October 2013 4:20 PM](#345584a13e392141f225cf6b99531a892961741d)

   I mean that if foo() and bar() are both top-level procedures (not lexically nested) and foo() is invoked as a coroutine, it can call bar() and bar() can yield for foo(). In Python this is not possible; in Lua, it is.

5. Brandon says:[1 November 2013 6:15 PM](#78be573cd7254f34556ec10d12124d8867829653)

   When was the decision made to go with shallow generators? I have to admit that python's generators are a little weak. Co-routines are more useful for organizing code (not forced to keep everything in one function) and for building abstractions on top of co-routines . I remember some discussion of this, but it seemed like you were on the fence about it. Fortunately cpython has the greenlet extension, which gives you true co-routines, but what about JS?

   Is support for full co-routines planned at some point in the future?

6. [Jeff Walden](http://whereswalden.com/) says:[8 November 2013 5:41 AM](#be6f6f52da69d4d43f97d0924643c2b67431a40a)

   I'm a bit late here, but I'd suggest `new Proxy({}, { get: ... })` instead of `Proxy.create(...)`, as the latter is a SpiderMonkey-ism that we plan to remove as ES6 direct proxies (the former) mature.

7. [Sean Halle](http://opensourceresearchinstitute.org) says:[1 February 2014 4:36 PM](#b3cbb6876a529673ddcd42f5afac4611aca38832)

   Hi,

   Thanks for these great articles. I'm just curious, this seems to relate to something I'm working on, which is adding parallel constructs to javascript.. or, a toolkit, actually, for rolling your own parallelism constructs..

   It has been a challenge figuring out whether any of the js engines are capable of functioning with multiple threads (the internal code cache, for example, must be protected from data races, along with similar bookkeeping structures inside the js engine). Does this co-routine ability imply that asynchronous routines can be run simultaneously in different threads?

   Thank, and thank you for the great posts!

   Sean

8. [Tim Caswell](http://creationix.com) says:[4 February 2014 4:45 PM](#3e2b08b7f7d5cc0f967f1073fdbf604db7d95484)

   @Brandon I don't remember the exact conversation about why shallow was chosen, but I wrote up a nice piece(1) about the tradeoffs between shallow and deep coroutines in JavaScript (we have both available in node.js thanks to C++ addons)

   My understanding was that shallow coroutines combined with yield\* is just as powerful as deep coroutines, but much more explicit and thus safer for less experienced developers.

   (1) http://howtonode.org/generators-vs-fibers

Comments are closed.
