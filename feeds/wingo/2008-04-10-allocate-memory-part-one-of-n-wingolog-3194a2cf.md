---
title: allocate, memory (part one of n) — wingolog
url: https://wingolog.org/archives/2008/04/10/allocate-memory-part-one-of-n
published: "2008-04-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/10/allocate-memory-part-one-of-n
---

# allocate, memory (part one of n) — wingolog

## [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)

10 April 2008 7:33 PM

- [nargery](/tags/nargery)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [garbage collection](/tags/garbage%20collection)
- [memory management](/tags/memory%20management)
- [emacs](/tags/emacs)
- [conservative](/tags/conservative)

In this article, I'd like to justify to myself in public a patch I've been cooking for a few months. It's a bit of [nargery](http://marnanel.livejournal.com/882917.html) (I love that word, thank you Thomas), but hey, I'm a bit of a narg. Well, maybe more than a bit: I'm even going to break this into two parts.

**in theory**

So one of the things that all high-level computer languages have in common is a garbage collector. (Notice how I implicitly define what an HLL is; sly, that.) What it means is that instead of having to manually allocate and free memory, you just allocate with wild abandon, and trust the computer to figure out what's junk and what's not.

There are a number of algorithms for this "junk detection" used in various systems, but most of them involve, at some stage, a process by which a set X of "live" objects is progressively enlarged to include all objects which are referenced by anything in set X. Objects which are not in that set are then thrown away.

So if I have a list in set X, eventually after running this "mark" phase, all of the elements of the list will also be in set X.

The trick is to determine what exactly is in set X at the start of the mark phase. This set is commonly called the "root set", because those objects form the roots of the live object tree. Global variables would belong to the root set, for example. Another important source of live object roots is the stack: the implicit chain of computations that is waiting to complete at any given moment, along with all of the resources needed to complete that computation.

**in practice**

So enough with the theory. Guile, being a Scheme interpreter designed to embed well with C, shares C's stack. It has no real idea how things are laid out on the stack itself, yet it somehow has to know which objects referenced on the stack are live Scheme objects (and not e.g. integers, C structures, etc.)

The strategy that Guile uses is known as "conservative" garbage collection. This means that it scans the memory region between the base of the stack and the current stack pointer, treating every word as if it were a Scheme object. If that word has the right [tag](http://www.gnu.org/software/guile/manual/html_node/Data-Representation-in-Scheme.html), and it points to other scheme objects in the heap, then it is deemed a part of the root set.

Obviously, other arrangements of bits, for example certain long integers, will also have this property. That is why this scheme (ha!) is known as "conservative", because "set X" is known to be a superset of the set of live objects. The worst that could happen would be a memory leak, not memory corruption.

The alternative to conservative marking is precise marking, like Python does with its refcounts, or like Emacs does with [explicitly adding values to the root set](http://www.gnu.org/software/emacs/manual/html_node/elisp/Writing-Emacs-Primitives.html) (via the GCPRO macros). But it's a lot of bookkeeping, and really [easy to make mistakes](//wingolog.org/archives/2005/06/17/up-to-the-date). Combine that with continuations and dynamic-wind and you have yourself a right mess.

**a pause**

And with that, I'll rest for a bit, and then see if I can continue tomorrow. Happy hacking!

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)
- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)
- [are ephemerons primitive?](/archives/2022/11/28/are-ephemerons-primitive)
- [ephemerons and finalizers](/archives/2022/10/31/ephemerons-and-finalizers)

Comments are closed.
