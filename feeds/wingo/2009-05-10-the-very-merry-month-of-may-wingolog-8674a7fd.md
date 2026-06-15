---
title: the very merry month of may — wingolog
url: https://wingolog.org/archives/2009/05/10/the-very-merry-month-of-may
published: "2009-05-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/05/10/the-very-merry-month-of-may
---

# the very merry month of may — wingolog

## [the very merry month of may](/archives/2009/05/10/the-very-merry-month-of-may)

10 May 2009 8:23 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [ilc](/tags/ilc)
- [los angeles](/tags/los%20angeles)

Time passes.

[![](//wingolog.org/photos//2009/04/04/dscn0706.normal.jpg)](//wingolog.org/photos/index.py/photos/3709?tag=la)

In my last dispatch, I left readers dangling with partial summaries of the International Lisp Conference over in Boston; the only real thing I wanted to write more about was Sussman's lightning talk about visualizing the shapes of programs. Thankfully, my colleague Jao [already shared his impressions](http://programming-musings.org/2009/03/29/sussmaniana/), with more depth and eloquence than I could.

[![](//wingolog.org/photos//2009/04/05/dscn0715.normal.jpg)](//wingolog.org/photos/index.py/photos/3718?tag=la)

Then from there Jao and I headed on to LA to work for a few weeks. There's lots to write about from the work side of things; I like the [gig](http://oblong.com/) quite a bit. More words at a later date.

[![](//wingolog.org/photos//2009/05/02/dscn0749.normal.jpg)](//wingolog.org/photos/index.py/photos/3752)

Lately it seems that whenever I'm silent for a while, internet-wise, it's because I'm hacking on something that's not quite finished. Not that many things can be finished, mind you; but there is a rhythm of escape and return that hacking has, that you go out on exploratory tangents, and then only weeks later return to "home" -- maybe with freshly caught hackfish, maybe with empty hands.

The hunt that I'm on right now is to integrate [psyntax](http://www.cs.indiana.edu/chezscheme/syntax-case/) into Guile. Psyntax is an implementation of syntax-case, a syntax expander for Scheme. It is a library, written in Scheme, which allows you to define macros for Scheme. If that doesn't make sense to you, perhaps we can regroup in another weblog post :)

But for the hacking audience out there, greets once more. Guile has had psyntax available as a library, `(ice-9 syncase)`, for more than a decade. But this solution has two problems: one, that you have to explicitly request its presence in your program. This used to be a slow process, though now with Guile's compiler it's close to instantaneous. The second problem was more serious: syncase macros were not hygienic with respect to modules.

At a high level, "hygiene" in a program means that you can determine the meaning of a program by looking at its source. Macros, as programs that rewrite programs, can work against hygiene, sneakily introducing new variables into the rewritten programs that were not in the original text.

People who write syntax expanders for Scheme -- the programs that run macros on the source text -- have been very concerned about hygiene, and psyntax is no exception. But psyntax didn't know about Guile's modules.

Consider a macro like this:

```
(define-syntax foo
  (syntax-rules ()
    ((_ x)
     (bar x))))

```

This expander specifies that all instances of `(foo something)` be replaced with `(bar something)`. But which `bar`? The reasonable answer to this is, the one in the module in which `foo` was defined. But Guile's psyntax integration instead chose the `bar` where `foo` was *used*.

This was because, 12 years ago or whatever, the psyntax expander had no conception of modules, so it had no way to know that the identifier that it was introducing in the output text -- `bar` \-\- came from a particular module.

Since then, psyntax actually grew a module system itself -- but since Guile already had a module system, Guile's copy of psyntax became a fork, because we couldn't include new releases of upstream psyntax. (Not that we had the hackpower to do it at that point.)

So making the psyntax expander hygienic with respect to modules was quite a task. In the end, though, I got it working, and Guile's psyntax expander now does the Right Thing(tm). Check out Guile from git to give it a run.

Incidentally, psyntax's current incarnation can be found over at [Abdulaziz' site](http://ikarus-scheme.org/r6rs-libraries/). There are a number of newer features that I need to backport from current psyntax, something we can get to with time. But our R6RS module system implementation will be independent from it.

**a rose by any other name**

The next obvious step was that once psyntax was working correctly, and loading speedily, we really should be using it throughout Guile. Scheme programmers expect syntax-rules and syntax-case to be there, and now that we can provide that nicely, we should do so.

I had other motivations for a tighter integration of psyntax. The recursive descent expander-cum-compiler that I had written duplicated many things that psyntax did, and expanded into a less orthogonal language than psyntax's output.

Plus, once I looked at starting to do optimizations, I realized that I was going to run into problems because of lexical scoping. Consider the following code:

```
(let ((square (lambda (x) (* x x))))
  (define * +)
  (square 10))
=> 100

```

One can trivially inline the `square` helper function, but a naive approach changes the meaning of the program:

```
(let ()
  (define * +)
  ((lambda (x) (* x x)) 10))
=> 20

```

The problem is that `(* x x)` in the first example did not have the same meaning as its occurrence in the second example.

The trick that most compiler writers use to get around this is just to rename all lexical variables. You know that `(lambda (x) (+ x 1))` is the same as `(lambda (y) (+ y 1))`. This property is known as alpha equivalence -- the names of lexical bindings don't matter.

Alpha equivalence can be exploited to avoid problems like the one above. First we give the lexically bound variables new, unique names:

```
(let ((a (lambda (b) (* b b))))
  (define c +)
  (a 10))
=> 100

```

Now the inlining works as expected:

```
(let ()
  (define c +)
  ((lambda (b) (* b b)) 10))
=> 100

```

If I was going to have to go through the bother of renaming all of the variables anyway, I might as well use psyntax, which renames all variables as part of its transformation.

I go into all this detail because the topic is somewhat perennial. I was reading a [paper about the Glasgow Haskell Compiler](https://eprints.kfupm.edu.sa/63927/1/63927.pdf) earlier this evening, and they actually avoid renaming, because *generating fresh names is too expensive for them*:

> If the compiler were written in an impure language, fresh names could be allocated by side effect, but GHC is written in Haskell, which does not have side effects. Using the trees of \[ARS94\] is the best solution we know of, but it still involves plumbing a tree of fresh names everywhere they might be needed. Worse, the fresh names usually aren't needed, but the tree is nevertheless built. This is deeply irritating: loads of allocation for no purpose whatsoever. Finally, even if we were not worried about performance, it is sometimes extremely painful to get the name supply to where it is needed.

Color me amused.

**but yes, back to the story**

Anyway, I had sufficient motivation, and thankfully a chunk of time in which to poke at it. My [syncase-in-boot-9](http://git.savannah.gnu.org/cgit/guile.git/log/?h=syncase-in-boot-9) branch has the results. I reimplemented defmacro in terms of syntax-case, which is pleasant.

It was quite a difficult hack, because it's at boot time when you don't even have modules -- yet you need to know something about modules, for hygienic purposes, and then modules get booted later... quite some quandaries debugging it. But it at least recompiles itself, and seems to run fine.

I'm holding off on merging it because I'm still running the old recursive-descent compiler on its output, whereas I need to write a compiler that operates directly on its output instead of treating it as unanalyzed Scheme. Maybe next week.

[![](//wingolog.org/photos//2009/03/27/dscn0704.normal.jpg)](//wingolog.org/photos/index.py/photos/3707)

Enough writing for today. Tomorrow, back to work again -- hopefully with all of this written, further updates will be easier to push out.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [international lisp conference 2009, day three, part 1](/archives/2009/03/25/international-lisp-conference-day-three-part-)
- [international lisp conference -- day two](/archives/2009/03/24/international-lisp-conference-day-two)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 2 responses

1. [Jake ashby](http://www.tybee500.com/) says:[14 May 2009 4:37 AM](#eb2cc41061a3bf1a3c1924fb45a53cb451460278)

   Just trying to get a feel for things here... Approximately how many units of hackpower does a fellow such as yourself produce in an average afternoon? Given that coffee supply is adequate and all hacking is counted as productive.

2. Mike Gran says:[21 May 2009 4:23 PM](#4eeedb00ae3ca767d2007c65723c7e0e9416fa5e)

   Hi Andy.

   You might recognize my name from some of the Guile mailing lists. I've been playing a bit with Unicode support for Guile. I actually live in South L.A. I hope you are surviving well in our fair city.

   Feel free to contact me should you need a hand when you're in town.

   -Mike

Comments are closed.
