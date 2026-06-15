---
title: cps soup — wingolog
url: https://wingolog.org/archives/2015/07/27/cps-soup
published: "2015-07-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/07/27/cps-soup
---

# cps soup — wingolog

## [cps soup](/archives/2015/07/27/cps-soup)

27 July 2015 2:43 PM

- [guile](/tags/guile)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [compilers](/tags/compilers)
- [contification](/tags/contification)
- [cse](/tags/cse)
- [closure optimization](/tags/closure%20optimization)
- [clojure](/tags/clojure)
- [bagwell](/tags/bagwell)
- [persistent data structures](/tags/persistent%20data%20structures)

Hello internets! This blog goes out to my long-time readers who have followed my saga hacking on Guile's compiler. For the rest of you, a little history, then the new thing.

In the olden days, [Guile](http://gnu.org/s/guile/) had no compiler, just an interpreter written in C. Around 8 years ago now, we ported Guile to compile to bytecode. That bytecode is what is currently deployed as Guile 2.0. For many reasons we wanted to upgrade our compiler and virtual machine for Guile 2.2, and the result of that was [a new continuation-passing-style compiler for Guile](http://wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile). Check that link for all the backstory.

So, I was going to finish documenting this intermediate language about 5 months ago, in preparation for making the first Guile 2.2 prereleases. But something about it made me really unhappy. You can catch some foreshadowing of this in my [article from last August on common subexpression elimination](http://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile); I'll just quote a paragraph here:

> In essence, the scope tree doesn't necessarily reflect the dominator tree, so not all transformations you might like to make are syntactically valid. In Guile 2.2's CSE pass, we work around the issue by concurrently rewriting the scope tree to reflect the dominator tree. It's something I am seeing more and more and it gives me some pause as to the suitability of CPS as an intermediate language.

This is exactly the same concern that Matthew Fluet and Stephen Weeks had back in 2003:

> Thinking of it another way, both CPS and SSA require that variable definitions dominate uses. The difference is that using CPS as an IL requires that all transformations provide a proof of dominance in the form of the nesting, while SSA doesn't. Now, if a CPS transformation doesn't do too much rewriting, then the partial dominance information that it had from the input tree is sufficient for the output tree. Hence tree splicing works fine. However, sometimes it is not sufficient.
>
> As a concrete example, consider common-subexpression elimination. Suppose we have a common subexpression `x = e` that dominates an expression `y = e` in a function. In CPS, if `y = e` happens to be within the scope of `x = e`, then we are fine and can rewrite it to `y = x`. If however, `y = e` is not within the scope of `x`, then either we have to do massive tree rewriting (essentially making the syntax tree closer to the dominator tree) or skip the optimization. Another way out is to simply use the syntax tree as an approximation to the dominator tree for common-subexpression elimination, but then you miss some optimization opportunities. On the other hand, with SSA, you simply compute the dominator tree, and can always replace `y = e` with `y = x`, without having to worry about providing a proof in the output that `x` dominates `y` (i.e. without putting `y` in the scope of `x`)
>
> [\[MLton-devel\] CPS vs SSA](http://mlton.org/pipermail/mlton/2003-January/023054.html)

To be honest I think all this talk about dominators is distracting. Dominators are but a lightweight flow analysis, and I usually find myself using [full-on flow analysis](http://wingolog.org/archives/2014/07/01/flow-analysis-in-guile) to compute the set of optimizations that I can do on a piece of code. In fact the only use I had for dominators in the nested CPS language was to rewrite scope trees! The salient part of Weeks' observation is that nested scope trees are the problem, not that dominators are the solution.

So, after literally years of hemming and hawing about this, I finally decided to remove nested scope trees from Guile's CPS intermediate language. Instead, a function is now a collection of labelled continuations, with one distinguished entry continuation. There is no more `$letk` term to nest continuations in each other. A program is now represented as a "soup" -- basically a map from labels to continuation bodies, again with a distinguished entry. As an example, consider this expression:

```
function(x):
  return add(x, 1)

```

If we rewrote it in continuation-passing style, we'd give the function a name for its "tail continuation", `ktail`, and annotate each expression with its continuation:

```
function(ktail, x):
  add(x, 1) -> ktail

```

Here the `-> ktail` means that the `add` expression passes its values to the continuation `ktail`.

With nested CPS, it could look like:

```
function(ktail, x):
  letk have_one(one): add(x, one) -> ktail
    load_constant(1) -> have_one

```

Here the label `have_one` is in a scope, as is the value `one`. With "CPS soup", though, it looks more like this:

```
function(ktail, x):
  label have_one(one): add(x, one) -> ktail
  label main(x): load_constant(1) -> have_one

```

It's a subtle change, but it took a few months to make so it's worth pointing out what's going on. The difference is that there is no scope tree for labels or variables any more. A variable can be used at a label if it flows to the label, in a flow analysis sense. Indeed, determining the set of variables that can be used at a label requires flow analysis; that's what Weeks was getting at in his 2003 mail about the advantages of SSA, which are really the advantages of an intermediate language without nested scope trees.

The question arises, though, now that we've decided on CPS soup, how should we represent a program as a value? We've gone from a nested term to a graph term, and we need to find a way to represent it somehow that facilitates looking up labels by name, and facilitates tree rewrites.

In Guile's IR, labels and variables are both integers, so happily enough, [we have such a data structure](http://wingolog.org/archives/2014/07/01/flow-analysis-in-guile): Clojure-style maps specialized for integer keys.

Friends, if there has been one realization or revolution for me in the last year, it has been Clojure-style data structures. Here's why. In compilers, I often have to build up some kind of analysis, then use that analysis to transform data. Often I need to keep the old term around while I build a new one, but it would be nice to share state between old and new terms. With a nested tree, if a leaf changed you'd have to rebuild all surrounding terms, which is gnarly. But with Clojure-style data structures, more and more I find myself computing in terms of values: build up this value, transform this map to that set, fold over this map -- and yes, you can fold over Guile's intmaps -- and so on. By providing an expressive data structure for which I can control performance characteristics by using transients if needed, these data structures make my programs more about data and less about gnarly machinery.

As a concrete example, the old contification pass in Guile, I didn't have the mental capacity to understand all the moving parts in such a way that I could compute an optimal contification from the beginning; instead I had to iterate to a fixed point, as Kennedy did in his "Compiling with Continuations, Continued" paper. With the new CPS soup language and with Clojure-style data structures, I could actually fit more of the algorithm into my head, with the result that Guile now contifies optimally while avoiding the fixed-point transformation. Also, the old pass used hash tables to represent the analysis, which I found incredibly confusing to reason about -- I totally buy Rich Hickey's argument that place-oriented programming is the source of many evils in programs, and hash tables are nothing if not a place party. Using functional maps let me solve harder problems because they are easier for me to reason about.

Contification isn't an isolated case, either. For example, we are able to do the *complete* set of optimizations from the "Optimizing closures in O(0) time" paper, including closure sharing, which I think makes Guile unique besides Chez Scheme. I wasn't capable of doing it on the old representation because it was just too hard for me to think about, because my data structures weren't right.

This new "CPS soup" language is still a first-order CPS language in that each term specifies its continuation, and that variable names appear in the continuation of a definition, not the definition itself. This effectively makes every variable a phi variable, in the sense of SSA, and you have to do some work to get to a variable's definition. It could be that still this isn't the right number of names; consider this function:

```
function foo(k, x):
  label have_y(y) bar(y) -> k
  label y_is_two() load_constant(2) -> have_y
  label y_is_one() load_constant(1) -> have_y
  label main(x) if x -> y_is_one else -> y_is_two

```

Here there is no distinguished name for the value `load_constant(1)` versus `load_constant(2)`: both are possible values for `y`. If we ended up giving them names, we'd have to reintroduce actual phi variables for the joins, which would basically complete the transformation to SSA. Until now though I haven't wanted those names, so perhaps I can put this off. On the other hand, every term has a label, which simplifies many things compared to having to contain terms in basic blocks, as is usually done in SSA. Yet another chapter in [CPS is SSA is CPS is SSA](http://wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers), it seems.

Welp, that's all the nerdery for right now. Talk at yall later!

## related articles

- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [doing it wrong: cse in guile](/archives/2012/05/14/doing-it-wrong-cse-in-guile)

### 3 responses

1. [Colin Fleming](https://cursiveclojure.com) says:[27 July 2015 4:10 PM](#fe21b9b6a55d37e827712cd36c8d135cd85920ee)

   Indeed, persistent data structures are wonderful things. They make many aspects of the language analysis that I do in Cursive much, much easier. Want to keep the state of your indexing at any point in the file? No problem, you already did. Maintaining state in backtracking PEG parsers? No problem, it comes for free.

   Lovely.

2. [Jaromil](http://www.dyne.org) says:[28 July 2015 0:07 AM](#85ab0b933fb705ce43fd43ac608f8c7dbedfd361)

   Way to go! Guile is extremely important to the GNU project and you are making the right choices. Clojure is an amazing language, very empowering: is wise to learn from it.

3. Anonymous says:[28 July 2015 2:30 PM](#a1f3d2a04a9cbdc76b861eec86628981ee183dc4)

   It sounds like a big relief to get that deeper understanding of SSA vs CPS and how encouraging that changing the IL led to those dramatic improvements in code clarity! Thank you for sharing this lesson learned and all this work going into improving guile.

Comments are closed.
