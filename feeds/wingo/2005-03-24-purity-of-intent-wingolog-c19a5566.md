---
title: purity of intent — wingolog
url: https://wingolog.org/archives/2005/03/24/98
published: "2005-03-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/03/24/98
---

# purity of intent — wingolog

## [purity of intent](/archives/2005/03/24/98)

24 March 2005 4:00 PM

- [gnome](/tags/gnome)
- [gstreamer](/tags/gstreamer)

**unsolicited shill**

A couple of interesting [kernel](http://lwn.net/Articles/128644/) [pages](http://lwn.net/Articles/128681/) on [this week's LWN](http://lwn.net/Articles/128059/). Wow. In my limited experience, I've never seen programming journalism as consistently well-written as LWN. Keep it up, folks.

**G\_GNUC\_CONST**

GTK+ has a dynamic type system whereby type id's are assigned at runtime, but don't change over the course of the execution of the program. The idiomatic way to do this is with `get_type` functions:

```
/* in the header */
GType foo_get_type(void) G_GNUC_CONST;

/* implementation */
GType
foo_get_type(void)
{
    static GType type = 0;
    if (type == 0)
        type = register_type(...);
    return type;
}

```

Here, the G\_GNUC\_CONST means that the function will always return the same value, so if it is called three times in a block of code, the compiler is free to call it only once and use the same result later on.

That's what we thought it meant, anyway. When working on the 0.9 branch of GStreamer, Wim was seeing some warnings about registering types twice, which means that two threads were calling uninitialized `get_type` functions at the same time. But, we had explicitly made sure to call these objects' `get_type` functions before starting threads, so that the static variables would be initialized:

```
static void
init_types (void)
{
    /* called from the main thread, before others are created */
    foo_get_type();
    ...
}

```

What had happened was that the compiler optimized away the only call to `foo_get_type` in the initializing function because the return value was not used. An `__attribute__((const))` function not only returns a constant value, but its output is solely determined by its arguments; it is not even allowed to look at global memory. `pure` functions ( `G_GNUC_PURE`) *are* allowed to look at global memory, but can't modify it (i.e., can't have side effects). Since what we wanted was the side effect of registering the type, neither `const` nor `pure` was appropriate. Removing the attribute solved the problem.

Moral of the story: don't count your chickens before they hatch, if your chickens are in fact `const` or `pure` functions and you're using gcc mumble mumble mumble.

## related articles

- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [dv capture](/archives/2005/10/05/dv-capture)
- [Suboptimality](/archives/2005/09/30/suboptimality)
- [rococo, and then rubble](/archives/2012/06/08/rococo-and-then-rubble)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [to guadec!](/archives/2011/08/03/to-guadec)

Comments are closed.
