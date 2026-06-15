---
title: allocate, memory (part 2 of n) — wingolog
url: https://wingolog.org/archives/2008/04/11/allocate-memory-part-of-n
published: "2008-04-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/11/allocate-memory-part-of-n
---

# allocate, memory (part 2 of n) — wingolog

## [allocate, memory (part 2 of n)](/archives/2008/04/11/allocate-memory-part-of-n)

11 April 2008 6:11 PM

- [guile](/tags/guile)
- [goops](/tags/goops)
- [metaobject protocol](/tags/metaobject%20protocol)

In my [last installment](//wingolog.org/archives/2008/04/10/allocate-memory-part-one-of-n), I started by saying something about a patch, then got lost in the vagaries of garbage collection. Hopefully in this writing product I can reign in the dangling pointers.

**bit twiddling**

If you are programming in an environment with a garbage collector, and you want to expose C resource to your managed environment, you will have to cooperate with the garbage collector. Specifically, you will need to know when an object becomes garbage, so that you can deallocate its resources.

In Guile, the historic way to do this is via a specific kind of boxed value, the "small object", or "SMOB". A SMOB is a double- or quad-word object whose first word is a tag designating the type of the object, and the rest is for C code to manipulate. SMOB types have to be registered with the Guile runtime, and have type-specific free, printing, and marking functions.

SMOBs are impoverished objects, however. There are only 8 bits in which to store the SMOB type, and they must be registered manually in C. It would be impossible to associate one SMOB type with each GType, for instance. So for [Guile-GNOME](http://www.gnu.org/software/guile-gnome/), which is where I'm really getting with all of this, you have to wrap C objects on two levels: one generic SMOB for GTypeInstances, and one more object-oriented wrapper that exposes the GType.

This two-level wrapping is ugly, confusing, and [bug-prone](http://thread.gmane.org/gmane.lisp.guile.user/6372). Thinking about this, and looking at the `(gnome gobject)` module with an eye to a stable, supportable interface, I wondered: is there not some other way to get free() notifications from the garbage collector?

As you might guess, the answer is yes. But to fully explain, in my most verbose fashion, we'll have to take a look at how objects are represented in the Lisp family of languages. The following discussion is specific to Guile, but shares fundamental characteristics with Common Lisp and other standard Lisp systems.

**objects in scheme**

Guile's object system, GOOPS, is derived from TinyCLOS, via [STklos](http://www.stklos.org/Doc/html/stklos-ref-8.html#STklos-Object-System). This includes a full meta-object protocol, so all details of e.g. instance, slot, and class allocation are fully extensible, while maintaining compilability. The following graphic describes the memory layout produced by the default allocation protocol:

[![click for the svg](//wingolog.org/pub/goops-layout.png)](//wingolog.org/pub/goops-layout.svg)

Instances contain two important words, one to point their classes, and one to point to a data array. The data array is as long as the number of Scheme objects that need be associated with the instance: the set of slots associated with the object. Of course, other slot allocation strategies are possible, for example storing slot values in a hash table as Python does by default.

By way of illustration, the process of slot access, via `(slot-ref foo 'bar)` goes like this:

1. Get the class of the object (dereferencing the vtable pointer)

2. Find the allocation information for the slot

   - It is part of the class' "slot-definitions" slot
3. If the slot is allocated in the data array, there is a fast-path array access

4. Otherwise the slot-definition provides getter and setter procedures for the slot

In effect, the class tells you how to read the instance. The allocation lookup process can be optimized: if the class of the object is known beforehand, either via inferencing or declaration, the lookup can be performed at compilation time. Otherwise, and more likely, an accessor can be made that compiles getters and setters on the first lookup.

**and then I was like, anyway..**

It turns out that two little-known facts enable us to shove C pointers into GOOPS objects. First, slots in the default data array may be allocated either as Scheme values (the normal case), or as raw, untagged words. The latter case allows us to put random C data in Scheme object without confusing the garbage collector or the interpreter. Second, classes have such a "raw word" slot containing a pointer to a free function that will be called when instances are freed, ostensibly to free the data array. However we can override this value to perform type-specific deallocation, and then free the data array.

This realization let me collapse the old memory layout:

[![click for the svg](//wingolog.org/pub/goops-layout-before.png)](//wingolog.org/pub/goops-layout-before.svg)

into this:

[![click for the svg](//wingolog.org/pub/goops-layout-after.png)](//wingolog.org/pub/goops-layout-after.svg)

The sum total is that now a wrapper for a GObject is just 3 words: the vtable and data pointers, and a 1-element array to hold the GTypeInstance\* pointer. That is, 12 or 24 bytes, for 32- or 64-bit words, respectively. (There are a couple of inefficiencies that increase this number of words to about 6, but those will go away with time.)

That patch took me about 3.5 months to bake fully, and is present in the newly-released [Guile-GNOME 2.15.97](http://thread.gmane.org/gmane.lisp.guile.user/6532). I haven't had time to update the online documentation yet, but the number of exported procedures and variables is down by about 100 or 150, with no loss in expressive power. I'm pretty pleased!

## related articles

- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [on-stack replacement in v8](/archives/2011/06/20/on-stack-replacement-in-v8)
- [class redefinition in guile](/archives/2009/11/09/class-redefinition-in-guile)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [ecmascript for guile](/archives/2009/02/22/ecmascript-for-guile)
- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)

### 3 responses

1. [Hans Petter Jansson](http://hpjansson.org/) says:[12 April 2008 5:13 AM](#ddfbfddf1f3815dc995b7b64c33bf62f6c4f4a52)

   Nice diagrams! What app did you use to make them?

2. [Marius Gedminas](http://gedmin.as) says:[12 April 2008 8:07 PM](#2aad0d1dad712a1433438e3ecee2aadffdc5d3b5)

   Posts like this is why I love reading Planet GNOME.

3. [wingo](http://wingolog.org/) says:[12 April 2008 8:59 PM](#334c2ecf2b4558bed953cbf2e5e0adf371d3dd08)

   Hans, I used the excellent Inkscape. It has a learning curve, but its tutorials (Help > Tutorials) are a worthy investment that get you going fairly quickly.

   Thank you Marius, you're very kind. Happy hacking!

Comments are closed.
