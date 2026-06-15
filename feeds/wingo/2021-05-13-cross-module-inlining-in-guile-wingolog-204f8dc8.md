---
title: cross-module inlining in guile — wingolog
url: https://wingolog.org/archives/2021/05/13/cross-module-inlining-in-guile
published: "2021-05-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2021/05/13/cross-module-inlining-in-guile
---

# cross-module inlining in guile — wingolog

## [cross-module inlining in guile](/archives/2021/05/13/cross-module-inlining-in-guile)

13 May 2021 11:25 AM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [inlining](/tags/inlining)
- [partial evaluation](/tags/partial%20evaluation)
- [peval](/tags/peval)
- [compilers](/tags/compilers)

Greetings, hackers of spaceship Earth! Today's missive is about cross-module inlining in Guile.

**a bit of history**

Back in the day... what am I saying? I always start these posts with loads of context. Probably you know it all already. 10 years ago, Guile's [partial evaluation](https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile) pass extended the [macro-writer's bill of rights](https://www.youtube.com/watch?v=LIEX3tUliHw) to Schemers of the Guile persuasion. This pass makes local function definitions free in many cases: if they should be inlined and constant-folded, you are confident that they will be. `peval` lets you write clear programs with well-factored code and still have good optimization.

The `peval` pass did have a limitation, though, which wasn't its fault. In Guile, modules have historically been a first-order concept: modules are a kind of object with a hash table inside, which you build by mutating. I speak crassly but that's how it is. In such a world, it's hard to reason about top-level bindings: what module do they belong to? Could they ever be overridden? When you have a free reference to `a`, and there's a top-level definition of `a` in the current compilation unit, is that the `a` that's being referenced, or could it be something else? Could the binding be mutated in the future?

During the Guile 2.0 and 2.2 cycles, we punted on this question. But for 3.0, we added the notion of [declarative modules](https://www.gnu.org/software/guile/docs/master/guile.html/Declarative-Modules.html#Declarative-Modules). For these modules, bindings which are defined once in a module and which are not mutated in the compilation unit are declarative bindings, which can be reasoned about lexically. We actually translate them to a form of `letrec*`, which then enables inlining via `peval`, [contification, and closure optimization](https://wingolog.org/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure) \-\- in descending order of preference.

The upshot is that with Guile 3.0, top-level bindings are no longer optimization barriers, in the case of declarative modules, which are compatible enough with historic semantics and usage that they are on by default.

However, module boundaries have still been an optimization barrier. Take [`(srfi srfi-1)`](https://www.gnu.org/software/guile/manual/html_node/SRFI_002d1-Constructors.html), a little utility library on lists. One definition in the library is `xcons`, which is `cons` with arguments reversed. It's literally `(lambda (cdr car) (cons car cdr))`. But does the compiler know that? Would it know that `(car (xcons x y))` is the same as `y`? Until now, no, because no part of the optimizer will look into bindings from outside the compilation unit.

**mr compiler, tear down this wall**

But no longer! Guile can now inline across module boundaries -- in some circumstances. This feature will be part of a future Guile 3.0.8.

There are actually two parts of this. One is the compiler can identify a set of ["inlinable" values](https://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/tree-il/inlinable-exports.scm;h=8ea5725f3c98ed92e9765c946f64bf30bada6251;hb=HEAD) from a declarative module. An inlinable value is a small copyable expression. A copyable expression has no identity (it isn't a fresh allocation), and doesn't reference any module-private binding. Notably, lambda expressions can be copyable, depending on what they reference. The compiler then extends the module definition that's residualized in the compiled file to include a little procedure that, when passed a name, will return the [Tree-IL](https://www.gnu.org/software/guile/docs/master/guile.html/Tree_002dIL.html#Tree_002dIL) representation of that binding. The design of that was a little tricky; we want to avoid overhead when using the module outside of the compiler, even relocations. See `compute-encoding` in that module for details.

With all of that, we can call `((module-inlinable-exports (resolve-interface '(srfi srfi-1))) 'xcons)` and get back the Tree-IL equivalent of `(lambda (cdr car) (cons car cdr))`. Neat!

The other half of the facility is the actual inlining. Here we lean on `peval` again, causing `` forms to trigger an [attempt to copy the term](https://git.savannah.gnu.org/gitweb/?p=guile.git;a=commitdiff;h=fd5cb457fb3a450b4b14eb89c8dbd764ba8df52e) from the imported module to the residual expression, limited by the same effort counter as the rest of `peval`.

The end result is that we can be absolutely sure that constants in imported declarative modules will inline into their uses, and fairly sure that "small" procedures will inline too.

**caveat: compiled imported modules**

There are a few caveats about this facility, and they are sufficiently sharp that I should probably fix them some day. The first one is that for an imported module to expose inlinable definitions, the imported module needs to have been compiled already, not loaded from source. When you load a module from source using the interpreter instead of compiling it first, the pipeline is optimized for minimizing the latency between when you ask for the module and when it is available. There's no time to analyze the module to determine which exports are inlinable and so the module exposes no inlinable exports.

This caveat is mitigated by [automatic compilation](https://www.gnu.org/software/guile/manual/html_node/Compilation.html), enabled by default, which will compile imported modules as needed.

It could also be fixed for modules by topologically ordering the module compilation sequence; this would allow some parallelism in the build but less than before, though for module graphs with cycles (these exist!) you'd still have some weirdness.

**caveat: abi fragility**

Before Guile supported cross-module inlining, there was only explicit inlining across modules in Guile, [facilitated by macros](https://www.gnu.org/software/guile/manual/html_node/Inlinable-Procedures.html). If you write a module that has a `define-inlinable` export and you think about its ABI, then you know to consider any definition referenced by the inlinable export, and you know by experience that its code may be copied into other compilation units. Guile doesn't automatically recompile a dependent module when a macro that it uses changes, currently anyway. Admittedly this situation leans more on culture than on tools, which could be improved.

However, with automatically inlinable exports, this changes. Any definition in a module could be inlined into its uses in other modules. This may alter the ABI of a module in unexpected ways: you think that module C depends on module B, but after inlining it may depend on module A as well. Updating module B might not update the inlined copies of values from B into C -- as in the case of `define-inlinable`, but less lexically apparent.

At higher optimization levels, even private definitions in a module can be inlinable; these may be referenced if an exported macro from the module expands to a term that references a module-private variable, or if an inlinable exported binding references the private binding. But these optimization levels are off by default precisely because I fear the bugs.

Probably what this cries out for is some more sensible dependency tracking in build systems, but that is a topic for another day.

**caveat: reproducibility**

When you make a fresh checkout of Guile from git and build it, the build proceeds in the following way.

Firstly, we build `libguile`, the run-time library implemented in C.

Then we compile a "core" subset of Scheme files at optimization level `-O1`. This subset should include the evaluator, reader, macro expander, basic run-time, and compilers. (There is a bootstrap evaluator, reader, and macro expander in C, to start this process.) Say we have source files S0, S1, S2 and so on; generally speaking, these files correspond to Guile modules M0, M1, M2 etc. This first build produces compiled files C0, C1, C2, and so on. When compiling a file S2 implementing module M2, which happens to import M1 and M0, it may be M1 and M0 are provided by compiled files C1 and C0, or possibly they are loaded from the source files S1 and S0, or C1 and S0, or S1 and C0.

The bootstrap build uses `make` for parallelism, with each compile process starts afresh, importing all the modules that comprise the compiler and then using them to compile the target file. As the build proceeds, more and more module imports can be "serviced" by compiled files instead of source files, making the build go faster and faster. However this introduces system-specific nondeterminism as to the set of compiled files available when compiling any other file. This strategy works because it doesn't really matter whether module M1 is provided by compiled file C1 or source file S1; the compiler and the interpreter implement the same language.

Once the compiler is compiled at optimization level `-O1`, Guile then uses that freshly built compiler to build everything at `-O2`. We do it in this way because building some files at `-O1` then all files at `-O2` takes less time than going straight to `-O2`. If this sounds weird, [that's because it is](https://wingolog.org/archives/2016/01/11/the-half-strap-self-hosting-and-guile).

The resulting build is reproducible... mostly. There is a bug in which some unique identifiers generated as part of the implementation of macros can be non-reproducible in some cases, and that disabling parallel builds seems to solve the problem. The issue being that `gensym` (or equivalent) might be called a different number of times depending on whether you are loading a compiled module, or whether you need to read and macro-expand it. The resulting compiled files are equivalent under alpha-renaming but not bit-identical. This is a bug to fix.

Anyway, at optimization level `-O1`, Guile will record inlinable definitions. At `-O2`, Guile will actually try to do cross-module inlining. We run into two issues when compiling Guile; one is if we are in the `-O2` phase, and we compile a module M which uses module N, and N is not in the set of "core" modules. In that case depending on parallelism and compile order, N may be loaded from source, in which case it has no inlinable exports, or from a compiled file, in which case it does. This is not a great situation for the reliability of this optimization. I think probably in Guile we will switch so that all modules are compiled at `-O1` before compiling at `-O2`.

The second issue is more subtle: inlinable bindings are recorded *after optimization of the Tree-IL*. This is more optimal than recording inlinable bindings before optimization, as a term that is not inlinable due to its size in its initial form may become small enough to inline after optimization. However, at `-O2`, optimization includes cross-module inlining! A term that is inlinable at `-O1` may become not inlinable at `-O2` because it gets slightly larger, *or vice-versa*: terms that are too large at `-O1` could shrink at `-O2`. We don't even have a guarantee that we will reach a fixed point even if we repeatedly recompile all inputs at `-O2`, because we allow non-shrinking optimizations.

I think this probably calls for a topological ordering of module compilation inside Guile and perhaps in other modules. That would at least give us reproducibility, provided we avoid the feedback loop of keeping around `-O2` files compiled from a previous round, even if they are "up to date" (their corresponding source file didn't change).

**and for what?**

People who have worked on inliners will know what I mean that a good inliner is like a combine harvester: ruthlessly efficient, a qualitative improvement compared to not having one, but there is a pointy end with lots of whirling blades and it's important to stop at the end of the row. You do develop a sense of what will and won't inline, and I think Dybvig's "Macro writer's bill of rights" encompasses this sense. Luckily people don't lose fingers or limbs to inliners, but inliners can maim expectations, and cross-module inlining more so.

Still, what it buys us is the freedom to be abstract. I can define a module like:

```
(define-module (elf)
  #:export (ET_NONE ET_REL ET_EXEC ET_DYN ET_CORE))

(define ET_NONE		0)		; No file type
(define ET_REL		1)		; Relocatable file
(define ET_EXEC		2)		; Executable file
(define ET_DYN		3)		; Shared object file
(define ET_CORE		4)		; Core file

```

And if a module uses my `(elf)` module and references `ET_DYN`, I know that the module boundary doesn't prevent the value from being inlined as a constant (and possibly unboxed, etc).

I took a look and on our usual microbenchmark suite, cross-module inlining doesn't make a difference. But that's both a historical oddity and a bug: firstly that the benchmark suite comes from an old Scheme world that didn't have modules, and so won't benefit from cross-module inlining. Secondly, Scheme definitions from the "default" environment that aren't explicitly recognized as primitives aren't inlined, as the `(guile)` module isn't declarative. (Probably we should fix the latter at some point.)

But still, I'm really excited about this change! Guile developers use modules heavily and have been stepping around this optimization boundary for years. I count 100 direct uses of `define-inlinable` in Guile, a number of them inside macros, and many of these are to explicitly hack around the optimization barrier. I really look forward to seeing if we can remove some of these over time, to go back to plain old `define` and just trust the compiler to do what's needed.

**by the numbers**

I ran a quick analysis of the modules include in Guile to see what the impact was. Of the 321 files that define modules, 318 of them are declarative, and 88 contain inlinable exports (27% of total). Of the 6519 total bindings exported by declarative modules, 566 of those are inlinable (8.7%). Of the inlinable exports, 388 (69%) are functions (lambda expressions), 156 (28%) are constants, and 22 (4%) are "primitives" referenced by value and not by name, meaning definitions like `(define head car)` (instead of re-exporting `car` as `head`).

On the use side, 90 declarative modules import inlinable bindings (29%), resulting in about 1178 total attempts to copy inlinable bindings. 902 of those attempts are to copy a `lambda` expressions in operator position, which means that `peval` will attempt to inline their code. 46 of these attempts fail, perhaps due to size or effort constraints. 191 other attempts end up inlining constant values. 20 inlining attempts fail, perhaps because a lambda is used for a value. Somehow, 19 copied inlinable values get elided because they are evaluated only for their side effects, probably to clean up `let`-bound values that become unused due to copy propagation.

All in all, an interesting endeavor, and one to improve on going forward. Thanks for reading, and catch you next time!

## related articles

- [partial evaluation in guile](/archives/2011/10/11/partial-evaluation-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### One response

1. bjoli says:[20 May 2021 6:13 AM](#30e90150ac28d233fc5153f5cc50a7771e2ef478)

   I want to take this moment to say goodbye to that unclean feeling you get when using macros to inline constants.

   You will not be missed.

Comments are closed.
