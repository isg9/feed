---
title: literate programming with guile-lib — wingolog
url: https://wingolog.org/archives/2004/07/25/literate-programming-with-guile-lib
published: "2004-07-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/07/25/literate-programming-with-guile-lib
---

# literate programming with guile-lib — wingolog

## [literate programming with guile-lib](/archives/2004/07/25/literate-programming-with-guile-lib)

25 July 2004 7:42 PM

- [scheme](/tags/scheme)

### Documentation drift

Every programmer knows this problem first-hand. Over time, the bind between documentation and documented will unravel. It is a fundamental property of the universe, and cannot be avoided.

However, it can be ameliorated somewhat by reducing points of duplication. If the documented thing can actually describe itself, it is much easier to keep the generated documentation up-to-date. Granted, the programmer still has to tell the object how to describe itself, but as that can be done in source code, the task becomes much more manageable.

To do this, we need a way to weave together objects and their documentation. Such a system must be built on the substrate of a semantic document language.

### Not your father's Texinfo

The semantic documentation problem has been solved already, with Texinfo. Texinfo has a long history within the GNU project and indeed is uniquely suited to documenting programs. It is concise, has a relevant and complete vocabulary, and has great indexing and cross-reference facilities.

Texinfo, in its text representation, is not easy to manipulate. Fortunately, the `(texinfo)` package in [guile-lib](http://ambient.2y.net/wingo/software/guile-lib/) can parse Texinfo into a native scheme representation. The output of the parser is [SXML](http://okmij.org/ftp/Scheme/SXML.html), a S-expression realization of XML. `(texinfo)` also provides modules to chunk a large document into a hierarchy, renderers to HTML and plain text, and indexing capabilities. `(texinfo)` brings Texinfo out of the 1980s and exposes it to programmatic construction and manipulation.

### Docweaving

Only procedures, via docstrings, are traditionally associated with documentation. For other objects one must fool around with the `documentation` object-property. Of course, it uglifies code to be setting object properties all over the place. [`(scheme documentation)`](http://ambient.2y.net/wingo/software/guile-lib/scheme-documentation/) defines some helper macros, among them `define-class-with-docs`, `define-variable-with-docs`, and the more general `define-docs`, that assist the programmer to cleanly document her definitions.

For example, to define a procedure, one can use:

```
(define (factorial n)
  "Compute the factorial of a number @var{n}. @var{n} should be a positive integer."
  (if (positive? n)
      (* n (factorial (1- n)))
      1))

```

Note that the docstring was marked up with Texinfo. A variable should be defined similarly:

```
(define-variable-with-docs animals
  "List of animals in the house. Do @strong{not} touch the crocodile."
  '(rabbit cat minx crocodile))

```

### Reflection

In neither of the above examples is it necessary to state the name of the object, its type, or even (in the case of the procedure) its arguments. Guile knows all of this. The `object-stexi-documentation` from [`(texinfo reflection)`](http://ambient.2y.net/wingo/software/guile-lib/texinfo-reflection/) will weave the runtime knowledge into the object documentation. It will yield its documentation in stexinfo, the SXML texinfo realization:

```
(object-stexi-documentation factorial)
==> (*fragment*
     (deffn (@ (name "factorial") (args "n"))
            "Compute the factorial of a number "
            (var "n") ". " (var "n")
            " should be a positive integer."))

```

Guile's knowledge is reflected in the fact that it used `deffn`, and that it gave the `name` and `args` attributes properly. The programmer's knowledge has been semantically parsed into stexinfo. Sweet.

Modules also need documentation. The Guile format for module documentation is the commentary, an area at the beginning of a document begun by `;;; Commentary:` and ended with `;;; Code:`.

`module-stexi-documentation`, also from `(texinfo reflection)`, will parse the commentary as texinfo. It then weaves in the documentation for all bindings exported by the module, ordered by type and by name. The result is a complete documentation page for the module, still expressed as `stexinfo`. The `(texinfo reflection)` documentation page linked to above is its output, although templatized for this site.

### Environment fertilization

Having documentation expressed as semantic markup allows for an extremely rich interactive environment. Let us look at some of the possibilities afforded to us.

The obvious way to use this information is to make `help` more informative. Unfortunately, the stock `help` isn't extensible. guile-lib's `(scheme session)` is a copy of the version in `ice-9` with some hooks added; hopefully the patches will be accepted upstream.

In any case, `(texinfo reflection)` extends `help` to generate `stexinfo` docs, and then to render them to plain text with `(texinfo plain-text)`. The output is much like that of `info`:

```
- Function: factorial n
     Compute the factorial of a number N. N should be a positive integer.

```

Module help can be accessed in the same way:

```
guile> (help (texinfo html))

(texinfo html)
**************

Overview
========

This module implements transformation from `stexi' to HTML. Note that the output
of `stexi->shtml' is actually SXML with the HTML vocabulary. This means that the
output can be further processed, and that it must eventually be serialized by
sxml->xml in manual (sxml-simple). References (i.e., the `@ref' family of
commands) are resolved by a "ref-resolver". See add-ref-resolver! for more
information.

Usage
=====

- Function: add-ref-resolver! proc
     Add PROC to the head of the list of ref-resolvers. PROC will be expected to
     take the name of a node and the name of a manual and return the URL of the
     referent, or `#f' to pass control to the next ref-resolver in the list.

     The default ref-resolver will return the concatenation of the manual name,
     `#', and the node name.

- Function: stexi->shtml tree
     Transform the stexi TREE into shtml, resolving references via
     ref-resolvers. See the module commentary for more details.

```

(WordPress is adding some wierd newlines to the above. It looks fine as plain text.)

### Crystallization

The system described above is dynamic in nature: it uses information available to guile only at runtime. Often, though, it is desirable to produce static documentation in another format, for example on the web or in print. guile-lib give you the tools to generate documentation, for example [my docs for guile-lib itself](http://ambient.2y.net/software/guile-lib/) (click on the SXML link at the bottom to see how it was done).

If you are building a graphical application with [guile-gnome](http://home.gna.org/guile-gnome/), you have the option of presenting Texinfo documentation to the user with a modern help browser ( [screenshot](http://ambient.2y.net/wingo/tmp/texinfo-browser.png)). You can show standalone Texinfo files, but as the native format for the help browser is `stexinfo`, you can also show documentation generated at runtime, including module documentation.

### Conclusions

All of the above add up to a rich interactive environment for programming. The level of work on your part is about the same, but by exposing your docs to runtime, semantic processing, you allow Guile to tie the pieces together into a coherent whole. Furthermore, the information you gain can be exported to more static formats, even to the point of allowing the print documentation to be generated entirely from the source code.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

Comments are closed.
