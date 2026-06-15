---
title: a whiff of whiffle — wingolog
url: https://wingolog.org/archives/2023/11/16/a-whiff-of-whiffle
published: "2023-11-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/11/16/a-whiff-of-whiffle
---

# a whiff of whiffle — wingolog

## [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)

16 November 2023 9:11 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [whiffle](/tags/whiffle)
- [primitives](/tags/primitives)
- [tagging](/tags/tagging)
- [types](/tags/types)
- [tail calls](/tags/tail%20calls)

A couple nights ago I [wrote about a superfluous Scheme
implementation](https://wingolog.org/archives/2023/11/14/whiffle-a-purpose-built-scheme) and promised to move on from sheepishly justifying my egregious behavior in my next note, and finally mention some results from this experiment. Well, no: I am back on my bullshit. Tonight I write about a couple of implementation details that discerning readers may find of interest: value representation, the tail call issue, and the standard library.

### what is a value?

As a Lisp, Scheme is one of the early “dynamically typed” languages. These days when you say “type”, people immediately think [propositions
as
types](https://www.pure.ed.ac.uk/ws/files/20001186/propositions_as_types.pdf), mechanized proof of program properties, and so on. But “type” has another denotation which is all about values and almost not at all about terms: one might say that [`vector-ref`](https://man.scheme.org/vector-ref.3scm) has a type, but it’s not part of a proof; it’s just that if you try to `vector-ref` a pair instead of a vector, you get a run-time error. You can imagine values as being associated with *type tags*: annotations that can be inspected at run-time for, for example, the sort of error that `vector-ref` will throw if you call it on a pair.

Scheme systems usually have a finite set of type tags: there are fixnums, booleans, strings, pairs, symbols, and such, and they all have their own tag. Even a Scheme system that provides facilities for defining new disjoint types ( `define-record-type` et al) will implement these via a secondary type tag layer: for example that all record instances are have the same primary tag, and that you have to retrieve their record type descriptor to discriminate instances of different record types.

Anyway. In Whiffle there are immediate types and heap types. All values have a low-bit tag which is zero for heap objects and nonzero for immediates. For heap objects, the first word of the heap object has tagging in the low byte as well. The [3-bit heap tag for pairs is chosen so that
pairs can just be two words, with no header
word](https://github.com/wingo/whiffle/blob/main/include/whiffle/types.h#L43-L44). There is another 3-bit heap tag for forwarded objects, which is used but the GC when evacuating a value. Other objects [put their heap tags in the low 8
bits of the first word](https://github.com/wingo/whiffle/blob/main/include/whiffle/types.h#L49-L57). Additionally there is a “busy” tag word value, used to prevent races when evacuating from multiple threads.

Finally, for generational collection of objects that can be “large” – the definition of large depends on the collector implementation, and is not nicely documented, but is more than, like, 256 bytes – anyway these objects might need to have space for a “remembered” bit in the object themselves. This is not the case for pairs but is the case for, say, vectors: even though they are prolly smol, they might not be, and they need space for a remembered bit in the header.

### tail calls

When I started Whiffle, I thought, let’s just compile each Scheme function to a C function. Since all functions have the same type, clang and gcc will have no problem turning any tail call into a proper tail call.

This intuition was right and wrong: at optimization level `-O2`, this works great. We don’t even do any kind of loop recognition / contification: loop iterations are tail calls and all is fine. (Not the most optimal implementation technique, but the assumption is that for our test cases, GC costs will dominate.)

However, when something goes wrong, I will need to debug the program to see what’s up, and so you might think to compile at `-O0` or `-Og`. In that case, somehow gcc does not compile to tail calls. One time while debugging a program I was flummoxed at a segfault during the `call` instruction; turns out it was just stack overflow, and the `call` was trying to write the return address into an unmapped page. For clang, I could use the [`musttail`
attribute](https://clang.llvm.org/docs/AttributeReference.html#musttail); perhaps I should, to allow myself to debug properly.

Not being able to debug at `-O0` with gcc is annoying. I feel like if GNU were an actual thing, we would have had the equivalent of a `musttail` attribute 20 years ago already. But it’s not, and we still don’t.

### stdlib

So Whiffle makes C, and that C uses some [primitives defined as inline
functions](https://github.com/wingo/whiffle/blob/main/include/whiffle/vm.h). Whiffle actually [lexically embeds user Scheme
code](https://github.com/wingo/whiffle/blob/main/module/whiffle/input.scm#L42-L45) with a [prelude](https://github.com/wingo/whiffle/blob/main/runtime/prelude.scm), having exposed a [set of
primitives](https://github.com/wingo/whiffle/blob/main/module/whiffle/primitives.scm) to that prelude and to user code. The assumption is that the compiler will open-code all primitives, so that the conceit of providing a primitive from the Guile compilation host to the Whiffle guest magically works out, and that any reference to a free variable is an error. This works well enough, and it’s similar to [what we currently do in
Hoot](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/compile.scm?ref_type=heads#L2592) as well.

This is a quick and dirty strategy but it does let us [grow the
language](https://www.youtube.com/watch?v=lw6TaiXzHAE) to something worth using. I think I’ll come back to this local maximum later if I manage to write about what Hoot does with modules.

### coda

So, that’s Whiffle: the Guile compiler front-end for Scheme, applied to an expression that prepends a user’s program with a prelude, in a lexical context of a limited set of primitives, compiling to very simple C, in which tail calls are just `return f(...)`, relying on the C compiler to inline and optimize and all that.

Perhaps next up: some results on using Whiffle to test Whippet. Until then, good night!

## related articles

- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)

### One response

1. Xoranth says:[17 November 2023 2:57 PM](#d27689d05cfc1e3646df00ce9f0e8b19438182b6)

   You can enable tail call optimization during debug builds by using -foptimize-sibling-calls.

   I.e.

   gcc -g -Og -foptimize-sibling-calls my *file.c -o my* file

Comments are closed.
