---
title: class redefinition in guile — wingolog
url: https://wingolog.org/archives/2009/11/09/class-redefinition-in-guile
published: "2009-11-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/11/09/class-redefinition-in-guile
---

# class redefinition in guile — wingolog

## [class redefinition in guile](/archives/2009/11/09/class-redefinition-in-guile)

9 November 2009 2:16 PM

- [guile](/tags/guile)
- [goops](/tags/goops)
- [scheme](/tags/scheme)
- [mop](/tags/mop)
- [object closure](/tags/object%20closure)

Yes hello hello!

Long-time readers will perhaps recall this diagram:

[![](//wingolog.org/pub/goops-layout.png)](//wingolog.org/archives/2008/04/11/allocate-memory-part-of-n)

*figure zero: things as they were.*

It comes from an [article](//wingolog.org/archives/2008/04/11/allocate-memory-part-of-n) describing how Guile represents its objects in memory, with particular concern for Guile-GNOME.

I was hacking on this code recently, and realized that this representation was not as good as it could be. Our switch to the BDW garbage collector lets us be more flexible with the size of our allocations, and so we can actually allocate the "slots" of an object inline with the object itself:

[![](//wingolog.org/pub/goops-inline-slots.png)](//wingolog.org/pub/goops-inline-slots.svg)

*figure one: things how maybe they could have been.*

Alas, during the hack, I discovered a stumbling block: that this representation doesn't allow for classes to be redefined.

**redefine a data type, what?**

Yes, Guile's object-oriented programming system (GOOPS) allows you to redefine the types of your data. It's OK! [CLOS](http://en.wikipedia.org/wiki/Common_Lisp_Object_System) lets you do this too; it's an old tradition. Redefining a class at runtime allows you to develop by incremental changes, without restarting your program.

Of course once you change a class, its instances probably need to change too, probably reallocating their slots. So we have to reintroduce the indirection -- but allowing for locality in the normal, non-redefined case. Like this:

[![](//wingolog.org/pub/goops-indirection.png)](//wingolog.org/pub/goops-indirection.svg)

*figure two: things how they are, almost.*

So updating an instance is as simple as swapping a pointer!

Almost.

Well, not really. This is really something that's unique to Lisp, as far as I can tell, and not very widely-known in the programming community, and hey, I didn't completely understand it -- so man, do I have a topic for a blog or what.

**step one: make a hole in the box**

The way redefinition works is that first you make a new class, then you magically swap the new for the old, then instances lazily update -- as they are accessed, they check that their class is still valid, and if not update themselves. It's involved, yo, so I made a bunch of pictures.

[![](//wingolog.org/pub/goops-class-redefinition-1.png)](//wingolog.org/pub/goops-class-redefinition-1.svg)

*figure three: class redefinition begins with defining a new class.*

So yeah, figure three shows the new class, lying in wait beside the old one. Then comes the magic:

[![](//wingolog.org/pub/goops-class-redefinition-2.png)](//wingolog.org/pub/goops-class-redefinition-2.svg)

*figure four: same identity, different state.*

What just happened here? Well we just swapped the vtable and data pointers in the old and new classes. For all practical purposes, the old class is the new class, and vice versa. All purposes except one, that is: `eq?`. The old class maintains its identity, so that any code that references it, in a hash table for example, will see the same object, but with new state.

The class' *identity* is the same, but its *state* has changed. That's the key thing to notice here.

Now we mark the old class's data as being out of date, and the next time its instances check their class... what? Here we reach another stumbling block. The old class has already has new state, so it is already fresh -- meaning that the instance will think nothing is wrong. It could be that the instance was allocated when its class declared two slots, but now the class says that instances have three slots. Badness, this; badness.

So what really needs to happen is for instances to point not to the identity of their classes, but to the *state* of their classes. In practice this means pointing directly to their slots. This is actually an efficiency win, as it removes an indirection for most use cases. Comme ça:

[![](//wingolog.org/pub/goops-class-redefinition-3.png)](//wingolog.org/pub/goops-class-redefinition-3.svg)

*figure five: instances actually point to class state, not class identity.*

As we see in the figure, a well-known slot in the class holds the redefinition information -- normally unset, but if the class is invalidated, it will allow the instance to know exactly which version of the class it is changing from and to.

[![](//wingolog.org/pub/goops-class-redefinition-4.png)](//wingolog.org/pub/goops-class-redefinition-4.svg)

*figure six: new equilibrium.*

And finally, figure six shows the new state of affairs -- in which slot access has been redirected for all redefined classes, and all of their instances, transitively.

**efficiency**

All in all, this is quite OK efficiency-wise. Instance data is normally local, and class data is one indirection away. A redefined instance will have nonlocal data, but hey, not much you can do otherwise, without a copying collector.

There is one efficiency hack worth mentioning. Accessors, [discussed in an earlier article](//wingolog.org/archives/2008/04/22/object-closure-and-the-negative-specification), don't need to check and see if their class is up to date or not. This is because they are removed from the old class and re-added to the new one as part of the redefinition machinery.

**summary**

Redefinition is complicated, but pretty neat.

**really, that's the summary?**

Yes.

## related articles

- [object closure and the negative specification](/archives/2008/04/22/object-closure-and-the-negative-specification)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)

Comments are closed.
