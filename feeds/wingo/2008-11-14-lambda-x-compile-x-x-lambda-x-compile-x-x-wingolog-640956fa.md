---
title: ((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x))) — wingolog
url: https://wingolog.org/archives/2008/11/14/lambda-x-compile-x-x-lambda-x-compile-x-x
published: "2008-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/11/14/lambda-x-compile-x-x-lambda-x-compile-x-x
---

# ((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x))) — wingolog

## [((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x)))](/archives/2008/11/14/lambda-x-compile-x-x-lambda-x-compile-x-x)

14 November 2008 11:01 PM

- [quine](/tags/quine)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [compilation](/tags/compilation)

Over the last couple days, I implemented a [generic tower of compilers in Guile](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=commit;h=b0b180d5227d76f5ca1e2f48b06f6d45195bd1f8).

So for example, you have Scheme as a source language, which defines a compiler to GHIL (a scheme-like intermediate language, simpler and without macros), which defines a compiler to GLIL (a lower-level language), which defines a compiler to object code (the byte sequence of VM code).

The compiler takes the source and target language as parameters, and does a depth-first search on the compiler graph to figure out how to get there from here, so to speak.

All well and good you say, but there is a wrinkle, in that sometimes you actually want the result of the computation that you just compiled. This is most clearly the case at the REPL:

```
scheme@(guile-user)> ,compile 42
Disassembly of #:

nlocs = 0  nexts = 0

   0    (make-int8 42)                  ;; 42
   2    (return)

scheme@(guile-user)> 42
$1 = 42

```

So in order not to pollute my delightfully generic compiler with special cases as to whether we want to actually execute the bytecode, I added a fake language, "value", adding a compiler from objcode to value.

I realized when writing about all of this that there is a subset of values that is valid Scheme, bending my language tower into a circle. I suspected a lurking quine, and after some poking and later [inspiration from the internet](http://www.madore.org/~david/computers/quine.html), I found this one:

```
  ((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x)))

```

Delightful, no?

## related articles

- [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [guile bar mitzvah](/archives/2008/11/02/guile-bar-mitzvah)
- [breaking silence, innocuously](/archives/2008/09/18/breaking-silence-innocuously)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 2 responses

1. [Marius Gedminas](http://gedmin.as) says:[15 November 2008 0:40 AM](#d4e9134c66fc0c460f8cc198a6845c4b808bdea6)

   Neat!

   Why depth-first rather than breadth-first? If you ever happen to have alternative compilation chains from language A to language B wouldn't you prefer the shorter one?

2. [wingo](http://wingolog.org/) says:[15 November 2008 7:46 PM](#56f95136d337ce7d823b7d58f7df3b6954046477)

   Heya Marius! Glad you like that too.

   Depth-first and breadth-first are obviously the same for 1 or 0 solutions. Currently all queries have 1 or 0 solutions, so in practice the difference is moot.

   The question comes in extensibility and prioritization: how can a future user extend the compiler so that their passes are chosen before the system passes?

   I can't think of a way to do that with breadth-first search, given that compilers are stored as an alist on source languages (e.g. the scheme language has a compiler to ghil, ghil has a compiler to glil, etc). With depth first search the solution is obvious, just cons your pass onto e.g. the ghil language object, and it will be chosen if there is a path from your sublanguage to the user's desired target language.

   Happy hacking!

Comments are closed.
