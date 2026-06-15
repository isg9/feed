---
title: javascript weakmaps should be iterable — wingolog
url: https://wingolog.org/archives/2024/08/19/javascript-weakmaps-should-be-iterable
published: "2024-08-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/08/19/javascript-weakmaps-should-be-iterable
---

# javascript weakmaps should be iterable — wingolog

## [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)

19 August 2024 8:13 PM

- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)
- [weakmap](/tags/weakmap)
- [gc](/tags/gc)
- [ephemerons](/tags/ephemerons)
- [finalization](/tags/finalization)
- [finalizers](/tags/finalizers)
- [iteration](/tags/iteration)
- [es6](/tags/es6)
- [igalia](/tags/igalia)

Good evening. Tonight, a brief position statement: it is a mistake for JavaScript’s `WeakMap` to not be iterable, and we should fix it.

### story time

A `WeakMap` associates a key with a value, as long as the key is otherwise reachable in a program. (It is an [ephemeron
table](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers).)

When `WeakMap` was added to JavaScript, back in the ES6 times, some implementors thought that it could be reasonable to implement weak maps not as a data structure in its own right, but rather as a kind of property on each object. Under the hood, adding an *key→value* association to a map *M* would set `key[M] = value`. GC would be free to notice dead maps and remove their associations in live objects.

If you implement weak maps like this, or are open to the idea of such an implementation, then you can’t rely on the associations being enumerable from the map itself, as they are instead spread out among all the key objects. So, ES6 specified `WeakMap` as not being [iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols); given a map, you can’t know what’s in it.

As with many things GC-related, non-iterability of weak maps then gained a kind of legendary status: the lore states that non-iterability preserves some key flexibility for JS implementations, and therefore it is good, and you just have to accept it and move on.

### dynamics

Time passed, and two things happened.

One was that this distributed `WeakMap` implementation strategy did not pan out; everyone ended up implementing weak maps as their own kind of object, and people use [an algorithm like the one Steve Fink described a
couple years
ago](https://blog.mozilla.org/sfink/2022/06/09/ephemeron-tables-aka-javascript-weakmaps/) to compute the *map×key⇒value* conjunction. The main original motivation for non-iterability was no longer valid.

The second development was [`WeakRef` and
`FinalizationRegistry`](https://github.com/tc39/proposal-weakrefs?tab=readme-ov-file#weakrefs-tc39-proposal), which expose some details of reachability as viewed by the garbage collector to user JS code. With `WeakRef` (and `WeakMap`), [you can build an iterable
`WeakMap`](https://github.com/tc39/proposal-weakrefs?tab=readme-ov-file#iterable-weakmaps).

(Full disclosure: I did work on ES6 and had a hand in `FinalizationRegistry` but don’t do JS language work currently.)

Thing is, your iterable `WeakMap` is strictly inferior to what the browser can provide: its implementation is extraordinarily gnarly, shipped over the wire instead of already in the browser, uses more memory, is single-threaded and high-latency (because `FinalizationRegistry`), and non-standard. What if instead as language engineers we just did our jobs and standardized iterability, as we do with literally every other collection in the JS standard?

Just this morning I wrote [yet another iterable
`WeakSet`](https://gitlab.com/spritely/guile-hoot/-/blob/main/reflect-js/reflect.js?ref_type=heads#L158-201) (which has all the same concerns as `WeakMap`), and while it’s sufficient for my needs, it’s not good (lacking prompt cleanup of dead entries), and by construction can’t be great (because it has to be redundantly implemented on top of `WeakSet` instead of being there already).

I am sympathetic to deferring language specification decisions to allow the implementation space to be explored, but when the exploration is done and the dust has settled, we shouldn’t hesitate to pick a winner: JS weak maps and sets should be iterable. Godspeed, brave TC39 souls; should you take up this mantle, you are doing the Lord’s work!

*Thanks to Philip Chimento for notes on the timeline and Joyee Cheung for notes on the iterable `WeakMap` implementation in the `WeakRef` spec. All errors* *mine, of course!*

## related articles

- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [are ephemerons primitive?](/archives/2022/11/28/are-ephemerons-primitive)
- [ephemerons and finalizers](/archives/2022/10/31/ephemerons-and-finalizers)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)

### 3 responses

1. Corbin says:[21 August 2024 2:52 PM](#64c447289ed8a7d33e26b5d30508501c51d96188)

   Hi Andy,

   WeakMap is the best rights-amplification primitive in ECMAScript, and it was intentionally non-iterable in order to preserve its suitability for that purpose. It wasn’t a mistake on their part, but a deliberate omission. I’ve left a link farm over at Lobsters with choice quotes from Mark Miller and the E authors, [here](https://lobste.rs/s/cdedna/javascript_weakmaps_should_be_iterable#c_w9bhv6).

2. Andre Wachsmuth says:[2 September 2024 1:42 PM](#a92a8bf6f263a7f2d9703efb50fe3aec8ce1759c)

   Thanks for the article, it was a good read! I needed an iterable weak set as well and the links were quite helpful for that. Would be nice if it were available natively. Another downside of having to implement it oneself is that it may not work exactly like a native Set works. For example, the way you remove entries from the #array changes the iteration order (for the native Set, the spec guarantess insertion order). Though that’s probably fine when used for a special purpose.

3. Andre Wachsmuth says:[2 September 2024 1:45 PM](#1c87b78887cd0b9082179d48f5e11b38fedb064e)

   If it helps anybody, this is my implementation of an iterable weak set

   https://pastebin.com/R10f1SrT

Comments are closed.
