---
title: i accidentally a scheme — wingolog
url: https://wingolog.org/archives/2023/11/13/i-accidentally-a-scheme
published: "2023-11-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/11/13/i-accidentally-a-scheme
---

# i accidentally a scheme — wingolog

## [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)

13 November 2023 9:36 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [whiffle](/tags/whiffle)
- [apologies](/tags/apologies)

Good evening, dear hackfriends. Tonight’s missive is an apology: not quite in the sense of expiation, though not quite *not* that, either; rather, apology in the sense of explanation, of exegesis: apologia. See, I accidentally made a Scheme. I know I have enough Scheme implementations already, but I went and made another one. It’s for a maybe good reason, though!

### one does not simply a scheme

I feel like we should make this the decade of leaning into your problems, and I have a Scheme problem, so here we are. See, I co-maintain [Guile](https://gnu.org/s/guile), and have been noodling on a new garbage collector (GC) for Guile, [Whippet](https://github.com/wingo/whippet). Whippet is designed to be embedded in the project that uses it, so one day I hope it will be just copied into Guile’s source tree, replacing the venerable [BDW-GC](https://github.com/ivmai/bdwgc) that we currently use.

The thing is, though, that GC implementations are complicated. A bug in a GC usually manifests itself far away in time and space from the code that caused the bug. Language implementations are also complicated, for similar reasons. Swapping one GC for another is something to be done very carefully. This is even more the case when the switching cost is high, which is the case with BDW-GC: as a collector written as a library to link into [“uncooperative”](https://hboehm.info/spe_gc_paper/) programs, there is more cost to moving to a conventional collector than in the case where the embedding program is already aware that (for example) garbage collection may relocate objects.

So, you need to start small. First, we need to [prove that the new GC
implementation is promising in some way](https://wingolog.org/archives/2023/02/07/whippet-towards-a-new-local-maximum), that it might improve on BDW-GC. Then... embed it directly into Guile? That sounds like a bug farm. Is there not any intermediate step that one might take?

But also, how do you actually test that a GC algorithm or implementation is interesting? You need a workload, and you need the ability to compare the new collector to the old, for that workload. In Whippet I had been writing some benchmarks in C ( [example](https://github.com/wingo/whippet/blob/main/benchmarks/mt-gcbench.c)), but this approach wasn’t scaling: besides not sparking joy, I was starting to wonder if what I was testing would actually reflect usage in Guile.

I had an early approach to rewrite a simple language implementation like the [other Scheme implementation I made to demonstrate JIT code
generation in
WebAssembly](https://wingolog.org/archives/2022/08/18/just-in-time-code-generation-within-webassembly), but that soon foundered against what seemed to me an unlikely rock: the compiler itself. In my [wasm-jit](https://github.com/wingo/wasm-jit/) work, the “compiler” itself was in C++, using the C++ allocator for compile-time allocations, and the result was a tree of AST nodes that were interpreted at run-time. But to embed the benchmarks in Whippet itself I needed something C, which is less amenable to abstraction of any kind... Here I think I could have made a different choice: to somehow allow C++ or something as a dependency to write tests, or to do more mallocation in the “compiler”...

But that wasn’t fun. A lesson I learned long ago is that if something isn’t fun, I need to turn it into a compiler. So I started writing a compiler to a little bytecode VM, initially in C, then in Scheme because C is a drag and why not? Why not just generate the bytecode C file from Scheme? Same dependency set, once the C file is generated. And then, as long as you’re generating C, why go through bytecode at all? Why not just, you know, generate C?

### after all, why not? why shouldn’t i keep it?

And that’s how I accidentally made a Scheme, [Whiffle](https://github.com/wingo/whiffle). Tomorrow I’ll write a little more on what Whiffle is and isn’t, and what it’s doing for me. Until then, happy hacking!

## related articles

- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)

### 2 responses

1. Peter U. says:[15 November 2023 9:53 AM](#a82d77413a3f16df0aadcc0e5e71993cee63f329)

   This is a test. 1234567890

   The quick brown fox jumps over the lazy dog. Bla bla..

2. [Adam](https://adamsolove.com) says:[15 November 2023 12:07 PM](#9b4a3316b85ec348ca93964654c56475bee4669c)

   I feel like we should make this the decade of leaning into your problems

   Said like a true addict, and I wholeheartedly agree. Love when the solution to a boring problem is a harder but much more interesting problem.

Comments are closed.
