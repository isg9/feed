---
title: abusing the c compiler — wingolog
url: https://wingolog.org/archives/2010/09/07/abusing-the-c-compiler
published: "2010-09-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/09/07/abusing-the-c-compiler
---

# abusing the c compiler — wingolog

## [abusing the c compiler](/archives/2010/09/07/abusing-the-c-compiler)

7 September 2010 8:43 PM

- [larceny](/tags/larceny)
- [scheme](/tags/scheme)
- [ffi](/tags/ffi)

**code reading**

Today I found something really neat in [Larceny](http://www.larcenists.org/)'s foreign function interface. The deal is that often times you need to parse a C structure or a preprocessor definition, and man, parsing C makes a body feel lazy. What's a hacker to do?

Larceny has an amusing take on this problem. The code looks straightforward enough:

```
;; parse out ent->d_name as a string
(define (dirent->name ent)
  (define-c-info (include<> "dirent.h")
    (struct "dirent" (name-offs "d_name")))
  (%peek-string (+ ent name-offs)))

```

The `define-c-info` block calculates `name-offs`, which is the offset of `d_name` in the `dirent` structure. `%peek-string` is something internal to Larceny that takes a memory address of a NUL-terminated C string and returns a Scheme string.

I had imagined, looking at this, that they had some kind of database of the headers and such, and in a sense they do -- *in the form of the C compiler*. `define-c-info` is a macro that runs the C compiler *at macro expansion time*, compiling and running a generated C program that spits out the relevant information as an s-expression on its stdout.

[![](//wingolog.org/pub/larceny-ffi.png)](http://www.ccs.neu.edu/home/pnkfelix/Published/klock-ffi-schemeworkshop-2008.html)

*some people like diagrams*

So in this case, if the `d_name` field starts 11 bytes into the structure, the generated C program will print out `(11)` on its stdout, and that number gets read in and inserted into the program. In that way `dirent->name` expands to something like:

```
(define (dirent->name ent)
  (define name-offs 11)
  (%peek-string (+ ent name-offs)))

```

Cool, no? The C compiler is only needed at compile-time, not at run-time.

Further details can be seen at Felix Klock's 2008 [paper on Larceny's FFI](http://www.ccs.neu.edu/home/pnkfelix/Published/klock-ffi-schemeworkshop-2008.html).

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [closedgl particle simulation in guile](/archives/2013/02/16/opengl-particle-simulation-in-guile)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [foreign interfaces](/archives/2007/05/24/foreign-interfaces)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)

### 2 responses

1. [jpc](http://loee.pl) says:[8 September 2010 10:00 AM](#5bccd09e2ca9cfe1abbb701422b2233fd16fa0b2)

   The first time I have seen the idea to use the C compiler like that was in Scheme->C \[1\] compiler (cdecl/sizeof.c). But to be precise it did it only once and afterwards used the collected sizeof+padding info.

   There is also a very neat idea like this in the Go implementation. \[2\] They parse the debugging information from the assembly output of the host compiler.

   \[1\]: https://alioth.debian.org/plugins/scmgit/cgi-bin/gitweb.cgi?p=scheme2c/scheme2c.git;a=blob;f=cdecl/sizeof.c

   \[2\]: http://golang.org/cmd/godefs/

2. [andrei](http://0xab.com) says:[8 September 2010 4:48 PM](#0acda511de3955af6928912e6f7b5e8a018f34c9)

   Although in Scheme->C that's a bit pointless and one day when I have time it will get simplified. Since we can inline C code we can just use the offsetof macro.

   http://0xab.com/code/matlab-scheme.git/blob/HEAD:/c-macros.sch#l87

Comments are closed.
