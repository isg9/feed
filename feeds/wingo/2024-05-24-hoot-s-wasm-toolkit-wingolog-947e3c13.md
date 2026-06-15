---
title: hoot's wasm toolkit — wingolog
url: https://wingolog.org/archives/2024/05/24/hoots-wasm-toolkit
published: "2024-05-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/05/24/hoots-wasm-toolkit
---

# hoot's wasm toolkit — wingolog

## [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

24 May 2024 10:37 AM

- [guile](/tags/guile)
- [hoot](/tags/hoot)
- [scheme](/tags/scheme)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [wasm](/tags/wasm)
- [webassembly](/tags/webassembly)
- [cps](/tags/cps)

Good morning! Today we continue our dive into [the Hoot
Scheme-to-WebAssembly compiler](https://spritely.institute/hoot). Instead of talking about Scheme, let’s focus on WebAssembly, specifically the set of tools that we have built in Hoot to wrangle Wasm. I am peddling a thesis: if you compile to Wasm, probably you should write a low-level Wasm toolchain as well.

(Incidentally, some of this material was taken from a [presentation I gave
to the Wasm standardization organization back in
October](https://wingolog.org/pub/wasm-cg-2023-10-scheme-slides.pdf), which I think I haven’t shared yet in this space, so if you want some more context, have at it.)

### naming things

Compilers are all about names: definitions of globals, types, local variables, and so on. An intermediate representation in a compiler is a graph of definitions and uses in which the edges are names, and the set of possible names is generally unbounded; compilers make more names when they see fit, for example when copying a subgraph via inlining, and remove names if they determine that a control or data-flow edge is not necessary. Having an unlimited set of names facilitates the graph transformation work that is the essence of a compiler.

Machines, though, generally deal with addresses, not names; one of the jobs of the compiler back-end is to tabulate the various names in a compilation unit, assigning them to addresses, for example when laying out an ELF binary. Some uses may refer to names from outside the current compilation unit, as when you use a function from the C library. The linker intervenes at the back-end to splice in definitions for dangling uses and applies the final assignment of names to addresses.

When targetting Wasm, consider what kinds of graph transformations you would like to make. You would probably like for the compiler to emit calls to functions from a [low-level run-time library written in
wasm](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/stdlib.scm?ref_type=heads#L106). Those functions are probably going to pull in some additional definitions, such as globals, types, exception tags, and so on. Then once you have your full graph, you might want to lower it, somehow: for example, you choose to use the [stringref](https://wingolog.org/archives/2023/10/19/requiem-for-a-stringref) string representation, but browsers don’t currently support it; you run a [post-pass](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/lower-stringrefs.scm) to lower to UTF-8 arrays, but then all your strings are not [constant](https://webassembly.github.io/gc/core/valid/instructions.html#constant-expressions), meaning they can’t be used as global initializers; so you run [another
post-pass to initialize globals in order from the start
function](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/lower-globals.scm). You might want to make other global optimizations as well, for example to [turn references to named locals into unnamed stack
operands](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/optimize.scm) (not yet working :).

Anyway what I am getting at is that you need a representation for Wasm in your compiler, and that representation needs to be fairly complete. At the very minimum, you need a facility to transform that in-memory representation to the standard [WebAssembly text
format](https://webassembly.github.io/gc/core/text/index.html), which allows you to use a third-party assembler and linker such as [Binaryen’s
`wasm-opt`](https://github.com/WebAssembly/binaryen). But since you have to have the in-memory representation for your own back-end purposes, probably you also implement the names-to-addresses mapping that will allow you to output [binary
WebAssembly](https://webassembly.github.io/gc/core/binary/index.html) also. Also it could be that Binaryen doesn’t support something you want to do; for example Hoot uses [block
parameters](https://wingolog.org/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n), which are supported fine in browsers but [not in
Binaryen](https://github.com/WebAssembly/binaryen/issues/6407).

(I exaggerate a little; Binaryen is a more reasonable choice now than it was before the GC proposal was stabilised. But it has been useful to be able to control Hoot’s output, for example as the exception-handling proposal has evolved.)

### one thing leads to another

Once you have a textual and binary writer, and an in-memory representation, perhaps you want to be able to read binaries as well; and perhaps you want to be able to read text. Reading the text format is a little annoying, but [I had implemented it already in
JavaScript](https://github.com/wingo/wassemble) a few years ago; and porting it to Scheme was a no-brainer, allowing me to easily author the run-time Wasm library as text.

And so now you have the beginnings of a full toolchain, built just out of necessity: reading, writing, in-memory construction and transformation. But how are you going to test the output? Are you going to require a browser? That’s gross. Node? Sure, we have to check against production Wasm engines, and that’s probably the easiest path to take; still, would be nice if this were optional. Wasmtime? But that doesn’t do GC.

No, of course not, you are a dirty little compilers developer, you are just going to implement a little wasm interpreter, aren’t you. Of course you are. That way you can [build nice debugging
tools](https://spritely.institute/files/docs/guile-hoot/0.4.1/Low_002dlevel-development-tools.html) to help you understand when things go wrong. Hoot’s interpreter doesn’t pretend to be high-performance—it is not—but it is simple and it just works. Massive kudos to Spritely hacker [David Thompson](https://dthompson.us/) for implementing this. I think implementing a Wasm VM also had the pleasant side effect that David is now a Wasm expert; implementation is the best way to learn.

Finally, one more benefit of having a Wasm toolchain as part of the compiler: `%inline-wasm`. In my example from last time, I had this snippet that makes a new bytevector:

```
(%inline-wasm
 '(func (param $len i32) (param $init i32)
    (result (ref eq))
    (struct.new
     $mutable-bytevector
     (i32.const 0)
     (array.new $raw-bytevector
                (local.get $init)
                (local.get $len))))
 len init)

```

`%inline-wasm` takes a literal as its first argument, which should parse as a Wasm function. Parsing guarantees that the wasm is syntactically valid, and allows the arity of the wasm to become apparent: we just read off the function’s type. Knowing the number of parameters and results is one thing, but we can do better, in that we also know their type, which we use for [intentional
types](https://nhplace.com/kent/PS/EQUAL.html), requiring in this case that the parameters be exact integers which get wrapped to the signed i32 range. The resulting term is spliced into the [CPS
graph](https://www.gnu.org/software/guile/manual/html_node/CPS-Soup.html), can be [analyzed for its side
effects](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/inline-wasm.scm#L40), and ultimately when written to the binary we [replace each local
reference in the Wasm with a reference of the appropriate local
variable](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/backend.scm#L733-803). All this is possible because we have the tools to work on Wasm itself.

### fin

Hoot’s Wasm toolchain is about 10K lines of code, and is fairly complete. I think it pays off for Hoot. If you are building a compiler targetting Wasm, consider budgetting for a 10K SLOC Wasm toolchain; you won’t regret it.

Next time, an article on Hoot’s use of CPS. Until then, happy hacking!

## related articles

- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

### One response

1. [hazn](https://hazn.me) says:[24 May 2024 3:45 PM](#ac7be22a322e23d38e0fa7cd95261f9f928a449c)

   Super cool, love your work on guile -> wasm!

   I know that wasm is used for ide extensions. i think hoot + [tree-sitter](https://github.com/tree-sitter/tree-sitter) would be a killer combo to develop next-gen ide extensions, a la [zed](https://zed.dev/blog/language-extensions-part-1)

   i’m a newbie to lisp, but i see tremendous potential in combining s-expressions with abstract syntax trees for a truly next gen ide experience. would love to hear your thoughts (i know that next-gen ide’s like emacs + geiser exist already for scheme)

Comments are closed.
