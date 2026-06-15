---
title: 'scheme modules vs whole-program compilation: fight — wingolog'
url: https://wingolog.org/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight
published: "2024-01-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight
---

# scheme modules vs whole-program compilation: fight — wingolog

## [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)

5 January 2024 8:43 PM

- [scheme](/tags/scheme)
- [modules](/tags/modules)
- [compilation](/tags/compilation)
- [guile](/tags/guile)
- [hoot](/tags/hoot)
- [spritely](/tags/spritely)
- [igalia](/tags/igalia)
- [whiffle](/tags/whiffle)
- [macros](/tags/macros)
- [psyntax](/tags/psyntax)

In a recent dispatch, I explained [the whole-program compilation
strategy used in Whiffle and
Hoot](https://wingolog.org/archives/2023/11/16/a-whiff-of-whiffle). Today’s note explores what a correct solution might look like.

### being explicit

Consider a module that exports an increment-this-integer procedure. We’ll use syntax from the [R6RS
standard](https://standards.scheme.org/corrected-r6rs/r6rs-Z-H-10.html#node_chap_7):

```
(library (inc)
  (export inc)
  (import (rnrs))
  (define (inc n) (+ n 1)))

```

If we then have a program:

```
(import (rnrs) (inc))
(inc 42)

```

Then the meaning of this program is clear: it reduces to `(+ 42 1)`, then to 43. Fine enough. But how do we get there? How does the compiler compose the program with the modules that it uses (transitively), to produce a single output?

In [Whiffle](https://wingolog.org/archives/2023/11/13/i-accidentally-a-scheme) (and [Hoot](https://gitlab.com/spritely/guile-hoot)), the answer is, sloppily. There is a [standard
prelude](https://github.com/wingo/whiffle/blob/main/runtime/prelude.scm) that initially has a number of bindings from the host compiler, [Guile](https://gnu.org/s/guile/). One of these is `+`, exposed under the name `%+`, where the `%` in this case is just a warning to the reader that this is a weird primitive binding. Using this primitive, the prelude defines a wrapper:

```
...
(define (+ x y) (%+ x y))
...

```

At compilation-time, Guile’s compiler recognizes `%+` as special, and therefore compiles the body of `+` as consisting of a primitive call ( *primcall*), in this case to the addition primitive. The Whiffle (and Hoot, and native Guile) back-ends then avoid referencing an imported binding when compiling `%+`, and instead produce backend-specific code: `%+` disappears. Most uses of the `+` wrapper get inlined so `%+` ends up generating code all over the program.

The prelude is lexically splatted into the compilation unit via a pre-expansion phase, so you end up with something like:

```
(let () ; establish lexical binding contour
  ...
  (define (+ x y) (%+ x y))
  ...
  (let () ; new nested contour
    (define (inc n) (+ n 1))
    (inc 42)))

```

This program will probably optimize (via partial evaluation) to just `43`. (What about `let` and `define`? Well. Perhaps we’ll get to that.)

But, again here I have taken a short-cut, which is about modules. Hoot and Whiffle don’t really do modules, yet anyway. I keep telling Spritely colleagues that it’s complicated, and rightfully they keep asking why, so this article gets into it.

### is it really a big `letrec`?

Firstly you have to ask, what is the compilation unit anyway? I mean, given a set of modules *A*, *B*, *C* and so on, you could choose to compile them separately, relying on the dynamic linker to compose them at run-time, or all together, letting the compiler gnaw on them all at once. Or, just *A* and *B*, and so on. One good-enough answer to this problem is [`library-group`](https://www.andykeep.com/pubs/scheme-10.pdf) form, which explicitly defines a set of topologically-sorted modules that should be compiled together. In our case, to treat the `(inc)` module together with our example program as one compilation unit, we would have:

```
(library-group
  ;; start with sequence of libraries
  ;; to include in compilation unit...
  (library (inc) ...)

  ;; then the tail is the program that
  ;; might use the libraries
  (import (rnrs) (inc))
  (inc 42))

```

In this example, the `(rnrs)` base library is not part of the compilation unit. Presumably it will be linked in, either as a build step or dynamically at run-time. For Hoot we would want the whole prelude to be included, because we don’t want any run-time dependencies. Anyway hopefully this would expand out to something like the set of nested `define` forms inside nested `let` lexical contours.

And that was my instinct: somehow we are going to smash all these modules together into a big nested `letrec`, and the compiler will go to town. And this would work, for a “normal” programming language.

But with Scheme, there is a problem: macros. Scheme is a “programmable programming language” that allows users to extend its syntax as well as its semantics. R6RS defines a procedural syntax transformer (“macro”) facility, in which the user can define functions that run on code at compile-time (specifically, during syntax expansion). Scheme macros manage to compose lexical scope from the macro definition with the scope at the macro instantiation site, by annotating these expressions with source location and scope information, and making syntax transformers mostly preserve those annotations.

“Macros are great!”, you say: well yes, of course. But they are a problem too. Consider this incomplete library:

```
(library (ctinc)
  (import (rnrs) (inc))
  (export ctinc)
  (define-syntax ctinc
    (lambda (stx)
      ...)) // ***

```

The idea is to define a version of `inc`, but at compile-time: a `(ctinc 42)` form should expand directly to `43`, not a call to `inc` (or even `+`, or `%+`). We define syntax transformers with `define-syntax` instead of `define`. The right-hand-side of the definition ( `(lambda (stx) ...)`) should be a procedure of one argument, which returns one value: so far so good. Or is it? How do we actually evaluate what `(lambda (stx) ...)` means? What should we fill in for `...`? When evaluating the transformer value, what definitions are in scope? What does `lambda` even mean in this context?

Well... here we butt up against the phasing wars of the mid-2000s. R6RS defines a [whole system to explicitly declare what bindings are
available
when](https://standards.scheme.org/corrected-r6rs/r6rs-Z-H-10.html#node_sec_7.2), then carves out a huge exception to allow for so-called [implicit
phasing](https://cs.indiana.edu/~dyb/pubs/implicit-phasing.pdf), in which the compiler figures it out on its own. In this example we imported `(rnrs)` for the default phase, and this is the module that defines `lambda` (and indeed `define` and `define-syntax`). The standard defines that `(rnrs)` makes its bindings available both at run-time and expansion-time (compilation-time), so `lambda` means what we expect that it does. Whew! Let’s just assume implicit phasing, going forward.

The operand to the syntax transformer is a *syntax object*: an expression annotated with source and scope information. To pick it apart, R6RS defines a pattern-matching helper, `syntax-case`. In our case `ctinc` is unary, so we can begin to flesh out the syntax transformer:

```
(library (ctinc)
  (import (rnrs) (inc))
  (export ctinc)
  (define-syntax ctinc
    (lambda (stx)
      (syntax-case stx ()
        ((ctinc n)
         (inc n)))))) // ***

```

But here there’s a detail, which is that when `syntax-case` destructures *stx* to its parts, those parts themselves are syntax objects which carry the scope and source location annotations. To strip those annotations, we call the `syntax->datum` procedure, exported by `(rnrs)`.

```
(library (ctinc)
  (import (rnrs) (inc))
  (export ctinc)
  (define-syntax ctinc
    (lambda (stx)
      (syntax-case stx ()
        ((ctinc n)
         (inc (syntax->datum #'n)))))))

```

And with this, voilà our program:

```
(library-group
  (library (inc) ...)
  (library (ctinc) ...)
  (import (rnrs) (ctinc))
  (ctinc 42))

```

This program should pre-expand to something like:

```
(let ()
  (define (inc n) (+ n 1))
  (let ()
    (define-syntax ctinc
      (lambda (stx)
        (syntax-case stx ()
          ((ctinc n)
           (inc (syntax->datum #'n))))))
    (ctinc 42)))

```

And then expansion should transform `(ctinc 42)` to 43. However, our naïve pre-expansion is not good enough for this to be possible. If you ran this in Guile you would get an error:

```
Syntax error:
unknown file:8:12: reference to identifier outside its scope in form inc

```

Which is to say, `inc` is not available as a value within the definition of `ctinc`. `ctinc` could residualize an expression that *refers* to `inc`, but it can’t use it to produce the output.

### modules are not expressible with local lexical binding

This brings us to the heart of the issue: with procedural macros, modules impose a phasing discipline on the expansion process. Definitions from any given module must be available both at expand-time and at run-time. In our example, `ctinc` needs `inc` at expand-time, which is an early part of the compiler that is unrelated to any later partial evaluation by the optimizer. We can’t make `inc` available at expand-time just using `let` / `letrec` bindings.

This is an annoying result! What do other languages do? Well, mostly they aren’t programmable, in the sense that they don’t have macros. There are some ways to get programmability using e.g. `eval` in JavaScript, but these systems are not very amenable to “offline” analysis of the kind needed by an ahead-of-time compiler.

For those declarative languages with macros, Scheme included, I understand the state of the art is to expand module-by-module and then stitch together the results of expansion later, using a kind of link-time optimization. You visit a module’s definitions twice: once to evaluate them while expanding, resulting in live definitions that can be used by further syntax expanders, and once to residualize an abstract syntax tree, which will eventually be spliced into the compilation unit.

Note that in general the expansion-time and the residual definitions don’t need to be the same, and indeed during cross-compilation they are often different. If you are compiling with Guile as host and Hoot as target, you might implement `cons` one way in Guile and another way in Hoot, choosing between them with `cond-expand`.

### lexical scope regained?

What is to be done? Glad you asked, Vladimir. But, I don’t really know. The compiler wants a big blob of `letrec`, but the expander wants a pearl-string of modules. Perhaps we try to satisfy them both? The `library-group` paper suggests that modules should be expanded one by one, then stitched into a `letrec` by AST transformations. It’s not that lexical scope is incompatible with modules and whole-program compilation; the problems arise when you add in macros. So by expanding first, in units of modules, we reduce high-level Scheme to a lower-level language without syntax transformers, but still on the level of `letrec`.

I was unreasonably pleased by the effectiveness of the “just splat in a prelude” approach, and I will miss it. I even pled for a kind of stop-gap fat-fingered solution to sloppily parse module forms and keep on splatting things together, but colleagues helpfully talked me away from the edge. So good-bye, sloppy: I repent my ways and will make amends, with 40 hail-maries and an alpha renaming thrice daily and more often if in moral distress. Further bulletins as events warrant. Until then, happy scheming!

## related articles

- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)

### One response

1. Sam Tobin-Hochstadt says:[8 January 2024 6:49 PM](#bd70b267fad14a3717beef0156eda5353f6bdc4c)

   You might be interested in how Racket solves this problem. Basically, Racket modules are compiled to a collection of *linklets*, which contain no macros themselves but do implement the relevant macros. Then given a particular linklet that’s the only one you’re interested in, you can flatten the collection into just the things that are actually necessary for the runtime behavior of the one you want. This is all part of how Racket bootstraps itself.

   You can see all the code starting here: https://github.com/racket/racket/blob/master/racket/src/expander/extract/main.rkt#L24

Comments are closed.
