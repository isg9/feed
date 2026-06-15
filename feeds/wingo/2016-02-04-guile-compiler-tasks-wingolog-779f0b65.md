---
title: guile compiler tasks — wingolog
url: https://wingolog.org/archives/2016/02/04/guile-compiler-tasks
published: "2016-02-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/02/04/guile-compiler-tasks
---

# guile compiler tasks — wingolog

## [guile compiler tasks](/archives/2016/02/04/guile-compiler-tasks)

4 February 2016 9:38 PM

- [guile](/tags/guile)
- [compilers](/tags/compilers)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [linkers](/tags/linkers)

Hey! We released Guile 2.1.2, including the [unboxing work](https://wingolog.org/archives/2016/01/19/unboxing-in-guile), and we fixed the [slow bootstrap problem](https://wingolog.org/archives/2016/01/11/the-half-strap-self-hosting-and-guile) by shipping [pre-built bootstraps in tarballs](http://git.savannah.gnu.org/cgit/guile.git/commit/?id=eccdeb6cc6c164ea10160fafbd276b6dc5e9b73e). A pretty OK solution in my opinion; [check it out!](http://lists.gnu.org/archive/html/guile-devel/2016-02/msg00001.html)

**future work**

At this point I think I'm happy with Guile's compiler and VM, enough for now. There is a lot more work to do but it's a good point at which to release a stable series. There will probably be a number of additional pre-releases, but not any more significant compiler/VM work that must be done before a release.

However, I was talking with Guilers at FOSDEM last weekend and we realized that although we do a pretty good job at communicating the haps in compiler-land, we don't do a good job at sharing a roadmap or making it possible for other folks to join the hack. And indeed, it's been difficult to do so while things were changing so much: I had to get things right in my head before joining in the confusion of other people's heads.

In that spirit I'd like to share a list of improvements that it would be nice to make at some point. If you take one of these tasks, be my guest: find me on IRC ( `wingo` on freenode) and let me know, and I'll help as I am able. You need to be somewhat independent; I'm not offering a proper mentoring or anything, more like office hours or something, where you come with the problem you are having and I commiserate and give context/background/advice as I am able.

So with that out of the way, here's a huge list of stuff! Following this, more details on each one.

1. stripping binaries

2. full source in binaries

3. cps in in binaries

4. linking multiple modules together

5. linking a single executable

6. instruction explosion

7. elisp optimizations

8. prompt removal

9. basic register allocation

10. optimal register allocation

11. unboxed record fields

12. textual CPS

13. avoiding arity checks

14. unboxed calls and returns

15. module-level inlining

16. cross-module inlining

As a bonus, in the end I'll give some notes on native compilation. But first, the hacks!

**stripping binaries**

Guile [uses ELF as its object file format](http://wingolog.org/archives/2014/01/19/elf-in-guile), and currently includes source location information as DWARF data. On space-constrained devices this might be too much. Your task: add a hack to the linker that can strip existing binaries. [Read Ian Lance Taylor's linker articles](http://wingolog.org/archives/2012/05/23/list-of-ian-lance-taylors-linker-articles) for more background, if you don't know things about linkers yet.

**full source in binaries**

Wouldn't it be nice if the ELF files that Guile generates actually included the source as well as the line numbers? We could do that, in a separate strippable ELF section. This point is like the reverse of the previous point :)

**cps in in binaries**

We could also include the [CPS IR](https://wingolog.org/archives/2015/07/27/cps-soup) in ELF files too. This would enable some kinds of link-time optimization and cross-module inlining. You'd need to define a binary format for CPS, like LLVM bitcode or so. Neat stuff :)

**linking multiple modules together**

Currently in Guile, just about every module is a separate `.go` file. Loading a module will cause a few `stat` calls and some seeks and reads and all that. Wouldn't it be nice if you could link together all the `.go` files that were commonly used into one object? Again this is a linker hack, but it needs support from the run-time as well: when the run-time goes to load a file, it should first check in a registry if that file has been logically provided by some other file. We'd be able to de-duplicate constant data from various modules. However there is an initialization phase when loading a `.go` file which effectively performs all the relocations needed by constants that need a fix-up at load-time; see the ELF article I linked to above for more. For some uses, it would be OK to produce one relocation/initialization procedure. For others, if you expected to only load a fraction of the modules in a `.go` file, it would be a lose on startup time,

so you would probably need to support lazy relocation when a module is first loaded.

Anyway, your task would be to write a linker hack that loads a bunch of `.go` files, finds the relocations in them, de-duplicates the constants, and writes out a combined `.go` file that includes a table of files contained in it. Good luck :) This hack would work *great* for Emacs, where it's effectively a form of `unexec` that doesn't actually rely on `unexec`.

**linking a single executable**

In the previous task, you could end up with the small `guile` binary that links to `libguile` (or your binary linking to `libguile`), and then a `.go` file containing all the modules you are interestd in. It sure would be nice to be able to link those together into just one binary, or at least to link the `.go` into the Guile binary. If the Guile is statically linked itself, you would have a statically linked application. If it's dynamically linked, it would remain dynamically linked. Again, a linker hack, but one that could provide a nicer way to distribute Guile binaries.

**instruction explosion**

Now we get more to the compiler side of things. Currently in Guile's VM there are instructions like [`vector-ref`](https://www.gnu.org/software/guile/docs/master/guile.html/Inlined-Scheme-Instructions.html#Inlined-Scheme-Instructions). This is a little silly: there are also instructions to branch on the type of an object ( [`br-if-tc7`](https://www.gnu.org/software/guile/docs/master/guile.html/Branch-Instructions.html#Branch-Instructions) in this case), to get the vector's length, and to do a branching integer comparison. Really we should replace `vector-ref` with a combination of these test-and-branches, with real control flow in the function, and then the actual ref should use some more primitive unchecked memory reference instruction. Optimization could end up hoisting everything but the primitive unchecked memory reference, while preserving safety, which would be a win. But probably in most cases optimization wouldn't manage to do

this, which would be a lose overall because you have more instruction dispatch.

Well, this transformation is something we need for native compilation anyway. I would accept a patch to do this kind of transformation on the `master` branch, after version 2.2.0 has forked. In theory this would remove most all high level instructions from the VM, making the bytecode closer to a virtual CPU, and likewise making it easier for the compiler to emit native code as it's working at a lower level.

**elisp optimizations**

Guile implements Emacs Lisp, and does so well. However it hasn't been the focus of a lot of optimization. Emacs has a lot of stuff going on on its side, and so have we, so we haven't managed to replace the Elisp interpreter in Emacs with one written in Guile, though Robin Templeton has brought us a long way forward. We need someone to do both the integration work but also to poke the compiler and make sure it's a clear win.

**prompt removal**

It's pretty natural to use delimited continuations when compiling some kind of construct that includes a `break` statement to Guile, whether that compiler is part of Elisp or just implemented as a Scheme macro. But, many instances of prompts can be contified, resulting in no overhead at run-time. Read up on contification and contify the hell out of some prompts!

**basic register allocation**

Guile usually tries its best to be safe-for-space: only the data which might be used in the future of a program is kept alive, and the rest is available for garbage collection. Notably, this applies to function arguments, temporaries, and lexical variables: if a value is dead, the GC can collect it and re-use its space. However this isn't always what you want. Sometimes you might want to have all variables that are in scope to be available, for better debugging. Your task would be to implement a "slot allocator" (which is really register allocation) that keeps values alive in the parts of the programs that they dominate.

**optimal register allocation**

On the other hand, our slot allocator -- which is basically register allocation, but for stack slots -- isn't so great. It does OK but you can often end up shuffling values in a loop, which is the worst. Your task would be to implement a proper register allocator: puzzle-solving, graph-coloring, iterative coalescing, *something* that really tries to do a good job. Good luck!

**unboxed record fields**

Guile's "structs", on which records are implemented, support unboxed values, but these values are untyped, not really integrated with the record layer, and always boxed in the VM. Your task would be to design a language facility that allows us to declare records with typed fields, and to store unboxed values in those fields, and to cause access to their values to emit boxing/unboxing instructions around them. The optimizer will get rid of those boxing/unboxing instructions if it can. Good luck!

**textual CPS**

The CPS language is key to all compiler work in Guile, but it doesn't have a nice textual form like LLVM IR does. Design one, and implement a parser and an unparser!

**avoiding arity checks**

If you know the procedure you are calling, like if it's lexically visible, then if you are calling it with the right number of arguments you can skip past the argument check and instead do a `call-label` directly into the body. Would be pretty neat!

**unboxed calls and returns**

Likewise if a function's callers are all known, it might be able to unbox its arguments or return value, if that's a good idea. Tricky! You could start with a type inference pass or so, and maybe that could produce some good debugging feedback too.

**module-level inlining**

Guile currently doesn't inline anything that's not lexically visible. Unfortunately this restriction extends to top-level definitions in a module: they are treated as mutable and so never inlined/optimized/etc. Probably we need to change the semantics here such that a module can be compiled as a unit, and all values which are never mutated can be assumed to be constant. Probably you also want a knob to turn off this behavior, but really you can always re-compile and re-load a module as a whole if re-loading a function at run-time doesn't work because it was inlined. Anyway. Some semantic work here, but some peval work as well. Be careful!

**cross-module inlining**

Likewise Guile currently doesn't inline definitions from other modules. However for small functions this really hurts. Guile should probably serialize tree-il for small definitions in `.go` files, and allow peval to speculatively inline imported definitions. This is related to the previous point and has some semantic implications.

**bobobobobobonus! native compilation**

Thinking realistically, native compilation is the next step. We have the object file format, cool. We will need the ability to call out from machine code in `.go` files to run-time functions, so we need to enhance the linker, possibly even with things like PLT/GOT sections to avoid dirtying too many pages. We need to lower the CPS even further, to get closer to some kind of machine model, then go specific, with an assembler for each architecture. The priority in the beginning will be simplicity and minimal complexity; good codegen will come later. This is obviously the most attractive thing but it's also the most tricky, design-wise. I want to do at least part of this, so though you can't have it all, you are welcome to help :)

That's it for now. I'll amend the post with more things as and when I think of them. Comments welcome too, as always. Happy hacking!

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [lightening run-time code generation](/archives/2019/05/24/lightening-run-time-code-generation)
- [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)

### One response

1. Johnny says:[3 May 2016 7:52 AM](#2e84e1d0392a9eda2f629d5ba375132afd122c1d)

   I had a 3x speedup for a macro expander for pascal. We have some crazy recursive macros with lots of precomputation. They were slow on 2.0, so I had a look around for different scheme impementations. A friend suggested trying the 2.1 branch, and you sure sprinkled some magic optimization dust over that. Thanks!

Comments are closed.
