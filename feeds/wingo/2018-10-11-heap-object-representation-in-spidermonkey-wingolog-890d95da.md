---
title: heap object representation in spidermonkey — wingolog
url: https://wingolog.org/archives/2018/10/11/heap-object-representation-in-spidermonkey
published: "2018-10-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/10/11/heap-object-representation-in-spidermonkey
---

# heap object representation in spidermonkey — wingolog

## [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)

11 October 2018 2:33 PM

- [spidermonkey](/tags/spidermonkey)
- [v8](/tags/v8)
- [jsc](/tags/jsc)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [javascript](/tags/javascript)
- [firefox](/tags/firefox)

I was having a look through SpiderMonkey's source code today and found something interesting about how it represents heap objects and wanted to share.

I was first looking to see how to implement [arbitrary-length integers ("bigints")](https://tc39.github.io/proposal-bigint/) by storing the digits inline in the allocated object. (I'll use the term "object" here, but from JS's perspective, bigints are rather *values*; they don't have identity. But I digress.) So you have a header indicating how many words it takes to store the digits, and the digits follow. This is how [JavaScriptCore](https://trac.webkit.org/browser/webkit/trunk/Source/JavaScriptCore/runtime/JSBigInt.cpp#L1551) and [V8](https://chromium.googlesource.com/v8/v8.git/+/master/src/objects/bigint.cc#1866) implementations of bigints work.

Incidentally, JSC's implementation was taken from V8. V8's was taken from Dart. Dart's was taken from Go. We might take SpiderMonkey's from Scheme48. Good times, right??

When seeing if SpiderMonkey could use this same strategy, I couldn't find how to make a variable-sized GC-managed allocation. It turns out that in SpiderMonkey you can't do that! SM's memory management system wants to work in terms of fixed-sized "cells". Even for objects that store properties inline in named slots, that's implemented in terms of standard cell sizes. So if an object has 6 slots, it might be implemented as instances of cells that hold 8 slots.

Truly variable-sized allocations seem to be managed off-heap, via malloc or other allocators. I am not quite sure how this works for GC-traced allocations like arrays, but let's assume that somehow it does.

Anyway, the point of this blog post. I was looking to see which part of SpiderMonkey reserves space for type information. For example, almost all objects in V8 start with a "map" word. This is the object's "hidden class". To know what kind of object you've got, you look at the map word. That word points to information corresponding to a class of objects; it's not available to store information that might vary between objects of that same class.

Interestingly, SpiderMonkey doesn't have a map word! Or at least, it doesn't have them on all allocations. Concretely, BigInt values don't need to reserve space for a map word. I can start storing data right from the beginning of the object.

But how can this work, you ask? How does the engine know what the type of some arbitrary object is?

The answer has a few interesting wrinkles. Firstly I should say that for objects that need hidden classes -- e.g. generic JavaScript objects -- there is indeed a map word. SpiderMonkey calls it a "Shape" instead of a "map" or a "hidden class" or a "structure" (as in JSC), but it's there, for that subset of objects.

But not all heap objects need to have these words. Strings, for example, are values rather than objects, and in SpiderMonkey they just have a small type code rather than a map word. But you know it's a string rather than something else in two ways: one, for "newborn" objects (those in the nursery), the [GC reserves a bit to indicate whether the object is a string or not](https://hg.mozilla.org/mozilla-central/file/tip/js/src/gc/Cell.h#l74). (Really: it's specific to strings.)

For objects promoted out to the heap ("tenured" objects), [objects of similar kinds are allocated in the same memory region](https://hg.mozilla.org/mozilla-central/file/tip/js/src/gc/Cell.h#l367) (in kind-specific "arenas"). There are about a dozen [trace kinds](https://hg.mozilla.org/mozilla-central/file/tip/js/public/TraceKind.h#l38), corresponding to arena kinds. To get the kind of object, you find its arena by rounding the object's address down to the arena size, then look at the arena to see what kind of objects it has.

There's another cell bit reserved to indicate that an object has been moved, and that the rest of the bits have been overwritten with a forwarding pointer. These two reserved bits mostly don't conflict with any use a derived class might want to make from the first word of an object; if the derived class uses the first word for integer data, it's easy to just reserve the bits. If the first word is a pointer, then it's probably always aligned to a 4- or 8-byte boundary, so the low bits are zero anyway.

The upshot is that while we won't be able to allocate digits inline to BigInt objects in SpiderMonkey in the general case, we won't have a per-object map word overhead; and we can optimize the common case of digits requiring only a word or two of storage to have the digit pointer point to inline storage. GC is about compromise, and it seems this can be a good one.

Well, that's all I wanted to say. Looking forward to getting BigInt turned on upstream in Firefox!

## related articles

- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [state of js implementations, 2014 edition](/archives/2014/12/09/state-of-js-implementations-2014-edition)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [inside javascriptcore's low-level interpreter](/archives/2012/06/27/inside-javascriptcores-low-level-interpreter)

### One response

1. [John Cowan](http://vrici.lojban.org/~cowan) says:[12 October 2018 0:48 AM](#c3b223599b75b4e6a4183a4b69cb9851ae76d988)

   Chicken used to use Scheme48 code, but has replaced it with native code. You can find this in the Chicken 4 numbers egg or as part of Chicken 5.

Comments are closed.
