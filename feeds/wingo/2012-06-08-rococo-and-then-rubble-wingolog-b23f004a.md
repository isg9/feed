---
title: rococo, and then rubble — wingolog
url: https://wingolog.org/archives/2012/06/08/rococo-and-then-rubble
published: "2012-06-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/06/08/rococo-and-then-rubble
---

# rococo, and then rubble — wingolog

## [rococo, and then rubble](/archives/2012/06/08/rococo-and-then-rubble)

8 June 2012 2:31 PM

- [elf](/tags/elf)
- [dwarf](/tags/dwarf)
- [gnome](/tags/gnome)
- [gir](/tags/gir)
- [introspection](/tags/introspection)

Why is it that we in GNOME came to view GLib as the lowest level of our stack?

Forget, for the moment, about such small barbarisms as `gint32`, and even `gint`. Forget things like gnome-keyring versus GnuPG. Let us even pass over GObject as a whole. Today I had a thought that was new to me. Let's talk about [DWARF](http://dwarfstd.org/).

DWARF describes ABIs. It is ubiquitous: it is on GNU systems as well as Mac and BSD, and could be on Windows if we put it there. It's extensible. It is mostly controlled by like-minded free software people.

So why is it that we invented [GIR](https://live.gnome.org/GObjectIntrospection)?

Discuss.

## related articles

- [elf in guile](/archives/2014/01/19/elf-in-guile)
- [dltool mines dwarf](/archives/2012/06/19/dltool-mines-dwarf)
- [compost, a leaf function compiler for guile](/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile)
- [list of ian lance taylor's linker articles](/archives/2012/05/23/list-of-ian-lance-taylors-linker-articles)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [to guadec!](/archives/2011/08/03/to-guadec)

### 13 responses

1. [wingo](http://wingolog.org) says:[8 June 2012 2:52 PM](#9939ffa3dd9e704d9c0009f273bc4b622edea005)

   Wisdom:

   dwarves and gnomes rarely get along

   there's the reason

2. Benjamin Otte says:[8 June 2012 2:53 PM](#310587e13400222dab0bf88face1e324b20eafb0)

   Conway's Law.

3. [Emmanuele Bassi](http://blogs.gnome.org/ebassi) says:[8 June 2012 3:47 PM](#a3c82190ecfe040db6d669ce3f39c95a008f94c4)

   because DWARF doesn't describe signals and properties, along with concepts of namespaces, enumeration and flag descriptions, that are kind of central to the whole concept of a project called "GObject Introspection". if you discount GObject as whole, then it's not weird that the point of the project is lost along with it.

4. [wingo](http://wingolog.org/) says:[8 June 2012 4:00 PM](#bf7b8b7ca6df7359322aa2cb9ea8dce3d1654dc2)

   Hi Emmanuele :)

   It is true that DWARF does not describe all of these, though it does have some support for namespaces and enumerated values. But my point was that it is extensible. Surely it is easier and simpler to add (e.g.) a GType attribute to an existing type DIE than it is to make a whole new toolchain.

5. [Colin Walters](http://blog.verbum.org) says:[8 June 2012 4:08 PM](#78c9fadcf247ff3ab04531aa9c60000dca814d1a)

   \> Why is it that we in GNOME came to view GLib as the lowest level

   of our stack?

   Because it has GMainContext, which is the fundamental tool for writing applications that don't suck (e.g. do async I/O).

   \> and could be on Windows if we put it there.

   That's a major reason. The second is that in order to gather all of the data, we need to build the library, and execute it. Then after we gather the data, we'd have to modify the library, which is just kind of gross.

   https://bugzilla.gnome.org/show\_bug.cgi?id=592311

   The third reason is that the separate typelib files in $libdir/girepository-1.0/Foo-1.0.typelib act as a lookup table, mapping "Foo-1.0" that gets typed in js/python into a .typelib file which then points to the .so to load.

   We wouldn't have to want to parse all .so files to find which one has Foo-1.0. (Though we could solve this with symlinks too I guess).

6. [Colin Walters](http://blog.verbum.org) says:[8 June 2012 4:09 PM](#8ac3c7a6f94cf86e6010ae6369e5109ecbc39e0e)

   Also, patches accepted...

7. wingo says:[8 June 2012 4:46 PM](#356801628f14a05799c7fd6f44677f8c5a8daea4)

   Thanks for stopping by, Colin, and for being gracious towards a troll ;-)

   So, just putting this out there: we can change the compiler. We can get GCC to emit arbitrary attributes on types and parameters, and place them into the binary that it compiles, in the .debug\_info (or into any other section). Depending on how it worked out things, one could avoid needing to load the library at build-time.

8. [Colin Walters](http://blog.verbum.org) says:[8 June 2012 6:33 PM](#281ba71f92f69663118a51b7d00f2ecef720fb70)

   Note that gobject-introspection came before GCC had plugins (there is also the question about tying an important part of the stack deeply to gcc).

   Anyways, certainly putting the typelib (as it exists today) inside the shared library as a separate section, and having a symlink in $libdir/girepository-1.0 would be kind of useful. Going beyond that would involve a lot of work; if you break libgirepository API you have a lot of porting to do...

   Approaching things from the other end and trying to kill g-ir-scanner in favor of a gcc plugin is far more interesting to me.

   But all of this is predicated on having someone actually do work...

9. Jasper St. Pierre says:[8 June 2012 8:38 PM](#2c58c4c6b054e4a7e8edac9e7ac0bb277c361bb2)

   GIR is meant to be an intermediate format that's easy to parse and emit, specialized towards what language bindings need to do. That's not what DWARF is about.

   Does DWARF have a simple mechanism of marking a char \* as being as string, or saying that it needs to be UTF-8, or that it needs to be in the native LANG-based locale? Grepping through the PDF, I don't see an answer to that.

10. wingo says:[8 June 2012 9:25 PM](#c1fa8b7480f1a5c5249b2def25d27b8077972a94)

    Hello Jasper. It is true that DWARF is a more general facility, but it is sufficient for the needs of GIR.

    For example, one could define a function as taking const char \* UTF8 str. UTF8 could be \_\_attribute\_\_((property("encoding", "utf-8"))) with new enough GCC. That could then get written into the .debug\_info as a type attribute that all could use: debuggers, bindings generators, etc. We would have to hack GCC to plumb through this information, but it is doable.

    Or, you could put type information into comments ;-)

11. Brendan Miller says:[8 June 2012 9:42 PM](#9c797b0e6a657a00855dfa0db6b929a519361355)

    The last time I checked, it wasn't so easy to consume or generate ELF and DWARF information. Is there a well documented, high quality library that is BSD or LGPL (not GPL)?

12. Logan says:[9 June 2012 0:43 AM](#fe3601646126dbf8f0016f5523147db90c768852)

    If we're throwing out alternatives to GIR, why not ECMA 335? Seems to cover the use case quite well and there's at least somewhat GNOME affiliated implementations.

13. wingo says:[9 June 2012 1:39 PM](#3948f941eeffca6b393fe22bd71bdfb03a2ff1b8)

    Hi Logan,

    Yours is a good question. I think it's fair to say that in GNOME, some of the solutions that we have implemented on the library level would be better made at the language and runtime level -- where by "runtime" I mean the language runtime. CIL is a good example of solutions at the language and runtime level.

    CIL is less interesting than ELF and DWARF (IMO) for libraries written in C, as it's not part of the toolchain for C.

    Finally I would note that I really respect the work that folks like Colin have put into GIR. We are better off having GIR than not. I just think it would be better implemented on another level, by digging down to lower layers like the compiler and the dynamic linker.

Comments are closed.
