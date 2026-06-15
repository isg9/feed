---
title: dltool mines dwarf — wingolog
url: https://wingolog.org/archives/2012/06/19/dltool-mines-dwarf
published: "2012-06-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/06/19/dltool-mines-dwarf
---

# dltool mines dwarf — wingolog

## [dltool mines dwarf](/archives/2012/06/19/dltool-mines-dwarf)

19 June 2012 3:55 PM

- [elf](/tags/elf)
- [dwarf](/tags/dwarf)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [debugging](/tags/debugging)

This is going to sound like quite a yak-shave, but here goes: I was spending some research time here at Igalia working on a new virtual machine for Guile when I got interested by [DWARF](http://dwarfstd.org/), the debugging format used in many UNIX systems (GNU, the BSDs, Mac OS). For various reasons, we needed a new debugging format, and DWARF seemed suitable.

I was going to need tools that would both produce and consume DWARF information, so I started with the parser. Of course there is a ready source of DWARF information on a GNU system, in all of the shared libraries. So I wrote a tool to snarf information out of those libraries. It is suprising how much there is!

**`dltool`**

Enough of the strange introduction! Let's try it out:

```
$ dltool print-one libc wmemchr
(subprogram
  (name "wmemchr")
  (prototyped #t)
  (type (pointer-type
          (byte-size 8)
          (type (named-type-reference (typedef "wchar_t")))))
  (formal-parameter
    (name "s")
    (type (pointer-type
            (byte-size 8)
            (type (const-type
                    (type (named-type-reference (typedef "wchar_t"))))))))
  (formal-parameter
    (name "c")
    (type (named-type-reference (typedef "wchar_t"))))
  (formal-parameter
    (name "n")
    (type (named-type-reference (typedef "size_t")))))

```

Nifty, no? It even prints the names of the formal parameters. The structure of the printout mirrors the structure of the DWARF information quite closely, so for a detailed reference you should refer to the DWARF standard. Note though that the "type" child of a "subprogram" (procedure, function, etc) denotes its return type. If it's missing, the function does not return a value.

In this case we see that `wmemchr` returns a pointer to a `wchar_t`, and we see that the pointer is 8 bytes wide. We don't see the definition of `wchar_t` though. You can pass a `--depth=N` to see more information, recursively, but it's most useful to simply have the tool print out a tree of needed declarations, this time with the `print-decls` command instead of `print-one`:

```
$ dltool print-decls libc wmemchr
(typedef
  (name "wchar_t")
  (type (base-type
          (byte-size 4)
          (encoding signed)
          (name "int"))))
(typedef
  (name "size_t")
  (type (base-type
          (byte-size 8)
          (encoding unsigned)
          (name "long unsigned int"))))
(subprogram
  (name "wmemchr")
  ...)

```

Here the body of the subprogram is the same as before. If I leave off the `wmemchr` argument, I get all the declarations that are publically exported by `libc`, and that have debugging information.

**locating debug information**

In the examples above, I'm just passing the basename of the library. `dltool` is smart enough to look in the library paths if needed, via parsing `/etc/ld.so.conf` and `$LD_LIBRARY_PATH`.

These days it is not so common for a stock GNU distribution to have debugging symbols installed for its libraries. Instead the debugging information is packaged separately, and a `.gnu_debuglink` section is left in the main binary (library, or executable, or both -- have you ran `/lib/ld-linux.so.2` lately?). `dltool` can deal with that.

The only thing it doesn't handle is compressed debug info ( `.zdebug_info` sections). Hopefully these sections go away at some point though: they prevent debuggers from mapping the debugging information directly, they don't save any distribution cost (packages being zipped already), and have no other runtime impact, as they debuginfo isn't normally loaded.

**c++?!?!**

Oddly enough, `dltool` works with C++ too. In this case, I point it at a debugging version of the JavaScriptCore library, looking for a definition of the `JSC::JSCell` class.

```
$ dltool print-one --grovel .libs/libjavascriptcoregtk-3.0.so JSC::JSCell
(class-type
  (specification
    (named-type-reference
      (structure-type "JSCell")
      (namespace "JSC")))
  (byte-size 16)
  (member
    (name "TypedArrayStorageType")
    (type (const-type
            (type (named-type-reference
                    (enumeration-type "TypedArrayType")
                    (namespace "JSC")))))
    (declaration #t)
    (const-value 0))
  (member
    (name "m_classInfo")
    (type (pointer-type
            (byte-size 8)
            (type (const-type
                    (type (named-type-reference
                            (structure-type "ClassInfo")
                            (namespace "JSC")))))))
    (data-member-location ((plus-uconst 0)))
    (accessibility private))
 ...)

```

In this case I used the `--grovel` option to indicate that I wanted to traverse all the debugging entries to find the one I wanted, instead of looking in the symbol table. That was necessary in this case because type definitions aren't present in a symbol table. The actual result is some 1500 lines long, with member variables and functions, and of course it's not complete as it's just a `print-one` output.

If the `data-member-declaration` entry looks a little odd to you, that's because it is odd: it's a sequence of opcodes for a virtual machine that computes locations of values. DWARF specifies a couple of little machines like this. For class members, there's usually only the one `plus-uconst` opcode, but with bitfields, different opcode sequences are possible.

**challenges**

Making `dltool` was quite an interesting hack. Oddly, writing the DWARF parser was the easy part. I had an ELF parser and linker already, and although quite flexible, DWARF is regular and well-specified.

One tricky part was making `dltool` fast, while not consuming too much memory. There can be a lot of data in a debugging object. For example, the debugging version of `libwebkitgtk-3.0.so` is bigger than 1GB. In a number of cases, we have to trade off the cost of re-parsing versus the cost of storing parsed data in memory. `dltool` can indeed handle WebKit, which is something I'm proud of, though printing all the exported declarations takes 2 GB of memory and some three or four minutes. More modest tasks like describing `libcairo` only take a second or two.

Another tricky part was determining which types are actually the same. In C and C++, typically you compile each file on its own, and then link them together. Each time you compile a file, the compiler loads the headers that the file needs and creates instances of those types, internal to the compiler. The debugging information that the compiler writes can contain as many (or more) definitions of (for example) `size_t` as there are compilation units that reference it. `dltool` needs to unify those types that are the same, not only to prevent multiple type declarations, but also to prevent divergence when processing recursive types.

C++'s nested namespaces also presented a problem. DWARF information is organized in trees, though they can also be reference each other by offset. However, there are no up-pointers in the tree nodes. It seems that you need to pre-traverse the node trees to determine the ranges of the namespaces so you can determine the right context for a node at any given offset. C does not have this issue, partly because it is less expressive, and partly due to some DWARF design decisions.

`dltool` doesn't currently demangle C++ names, for what that's worth.

Oddly, C++ templates were not very much of a problem. They simply create debugging entries with really crazy names, that's all.

**"automatic" ffi?**

I think `dltool` is pretty cool. Given an opaque information-ful object, I always want some tool to spread information on a screen in a structured way. So it's an interesting tool for binary spelunkers.

There are some obvious applications, though. One is to use the information that dltool finds to automatically generate bindings from C (or even C++) libraries to your language of choice. You can use a `dltool` run to generate the bindings at "compile-time", or save it to a file and generate bindings at run-time with node-ffi or ctypes or whatever the kids use these days.

Of course in Guile, besides importing the `dlhacks` module and creating bindings at runtime, we can get the best situation by running the dltool groveller from within a macro. That way we pull the DWARF groveller into our language, without having the module depend on dlhacks or on the presence of debugging information at runtime.

There are some limitations, though. DWARF is a flexible and extensible format, but it doesn't *currently* get all of the information you would want in a binding. For example there's no way to tell the C compiler to serialize an association between "struct ClutterActor" and a GType named "ClutterActor". You can't tell it that the return value of `wcsdup` is actually owned by the caller. You can't annotate pointer arguments to indicate that they are "out" or "in/out" or whatever. But this is not a failing of DWARF: it is a current limitation of our toolchain.

Actually it's also a limitation of C -- I mean, everything you would want to say *about* a variable or a function or whatever is, loosely speaking, the *type* of that object. We can't really change C's type system, but we can improve our toolchain by allowing for GCC to recognize and plumb through more relevant `__attribute__` annotations.

**all right, all ready**

Anyway, that's the thing! The tool itself requires Guile 2.0. If you're running Debian, just install guile-2.0, then:

```
git clone git://gitorious.org/guile-dlhacks/guile-dlhacks.git
cd guile-dlhacks
autoreconf -vif && ./configure && make
./env dltool help

```

Visit [the gitorious page](https://gitorious.org/guile-dlhacks) for more, and find me on `#guile` for more.

Happy spelunking!

## related articles

- [elf in guile](/archives/2014/01/19/elf-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

### 3 responses

1. D Herring says:[21 June 2012 1:21 AM](#a8319fa0274905c119ddbf0e20d925ef0674f599)

   I was playing with CL bindings to libdwarf a few years ago with the hopes of writing an auto-ffi. (libdwarf handles the different non-gnu variants of dwarf as well) Abandoned it after realizing how far it was from what I wanted. Debug files are huge (multiple GB for Qt), contain tons of redundant info, are hard to cross-reference, are hard to find, etc.

   I'm now well on my way to having a standards-compliant C++ preprocessor written in CL, with the thought of auto-generating bindings from header files. (No, I don't like SWIG or gccxml.)

2. Brandon says:[21 June 2012 11:35 PM](#ab73cc0a8affe986973cdb0dcbdf89bac15af7dd)

   I am curious how ELF and DWARF compare with the formats used for other run-time environments, such as Java and .NET. It seems that java .class files contain quite a bit of meta-data that compilers are able to extract and use.

3. Brandon says:[26 June 2012 4:38 PM](#db0fc1849a70401aaf90fd35407eed2472cbd065)

   I did some investigation on my own and discovered that the java specification requires that references to public symbols be represented symbolically in the compiler output. These references are resolved during initialization. Several sources have referred to this as 'late binding', in contrast with the 'early binding' of C and C++'s.

Comments are closed.
