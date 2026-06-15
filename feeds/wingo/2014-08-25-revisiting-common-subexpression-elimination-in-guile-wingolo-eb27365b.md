---
title: revisiting common subexpression elimination in guile — wingolog
url: https://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile
published: "2014-08-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile
---

# revisiting common subexpression elimination in guile — wingolog

## [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)

25 August 2014 9:48 AM

- [guile](/tags/guile)
- [compilers](/tags/compilers)
- [cse](/tags/cse)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)
- [effects analysis](/tags/effects%20analysis)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [scalar replacement](/tags/scalar%20replacement)

A couple years ago I wrote about a [common subexpression pass that I implemented in Guile 2.0](//wingolog.org/archives/2012/05/14/doing-it-wrong-cse-in-guile).

To recap, Guile 2.0 has a global, interproceduralcommon subexpression elimination (CSE) pass.

In the context of compiler optimizations, "global" means that it works across basic block boundaries. Basic blocks are simple, linear segments of code without control-flow joins or branches. Working only within basic blocks is called "local". Working across basic blocks requires some form of understanding of how values can flow within the blocks, for example [flow analysis](//wingolog.org/archives/2014/07/01/flow-analysis-in-guile).

"Interprocedural" means that Guile 2.0's CSE operates across closure boundaries. Guile 2.0's CSE is "context-insensitive", in the sense that *any* possible effect of a function is considered to occur at *all* call sites; there are newer CSE passes in the literature that separate effects of different call sites ("context-sensitive"), but that's not a Guile 2.0 thing. Being interprocedural was necessary for Guile 2.0, as its intermediate language could not represent (e.g.) loops directly.

The conclusion of my previous article was that although CSE could do cool things, in Guile 2.0 it was ultimately limited by the language that it operated on. Because the Tree-IL direct-style intermediate language didn't define order of evaluation, didn't give names to intermediate values, didn't have a way of explicitly representing loops and other kinds of first-order control flow, and couldn't precisely specify effects, the results, well, could have been better.

I know you all have been waiting for the last 27 months for an update, probably forgoing meaningful social interaction in the meantime because what if I posted a followup while you were gone? Be at ease, fictitious readers, because that day has finally come.

**CSE over CPS**

[The upcoming Guile 2.2 has a more expressive language for the optimizer to work on](//wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile), called continuation-passing style (CPS). CPS explicitly names all intermediate values and control-flow points, and can integrate nested functions into first-order control-flow via "contification". At the same time, [the Guile 2.2 virtual machine no longer penalizes named values](//wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile), which was another weak point of CSE in Guile 2.0. Additionally, [the CPS intermediate language enables more fined-grained effects analysis](//wingolog.org/archives/2014/05/18/effects-analysis-in-guile).

All of these points mean that CSE has the possibility to work better in Guile 2.2 than in Guile 2.0, and indeed it does. The shape of the algorithm is a bit different, though, and I thought some compiler nerds might be interested in the details. I'll follow up in the next section with some things that new CSE pass can do that the old one couldn't.

So, by way of comparison, the old CSE pass was a once-through depth-first visit of the nested expression tree. As the visit proceeded, the pass built up an "environment" of available expressions -- for example, that `(car a)` was evaluated and bound to `b`, and so on. This environment could be consulted to see if a expression was already present in the environment. If so, the environment would be traversed from most-recently-added to the found expression, to see if any intervening expression invalidated the result. Control-flow joins would cause recomputation of the environment, so that it only held valid values.

This simple strategy works for nested expressions without complex control-flow. CPS, on the other hand, can have loops and other control flow that Tree-IL cannot express, so for it to build up a set of "available expressions" requires a full-on [flow analysis](//wingolog.org/archives/2014/07/01/flow-analysis-in-guile). So that's what the pass does: a flow analysis over the labelled expressions in a function to compute the set of "available expressions" for each label. A labelled expression *a* is available at label *b* if *a* dominates *b*, and no intervening expression could have invalidated the results. [An expression invalidates a result if it may write to a memory location that the result may have read](//wingolog.org/archives/2014/05/18/effects-analysis-in-guile). The code, such as it is, may be found [here](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/cse.scm;h=3a03ede83aa4da7d778e1eabf45892db7640bda5;hb=HEAD#l64).

Once you have the set of available expressions for a function, you can proceed to the elimination phase. First, you start by creating an "eliminated variable" map, which initially maps each variable to itself, and an "equivalent expressions" table, which maps "keys" to a set of labels and bound variables. Then you visit each expression in a function, again in topologically sorted order. For each expression, you compute a "key", which is some unique representation of an expression that can be compared by structural equality. Keys that compare as equal are equivalent, and are subject to elimination.

For example, consider a call to the `add` primitive with variables labelled *b* and *c* as arguments. Imagine that *b* maps to *a* in the eliminated variable table. The expression as a whole would then have a key representation as the list `(primcall add a c)`. If this key is present in the equivalent expression table, you check to see if any of the equivalent labels is *available* at the current label. If so, hurrah! You mark the outputs of the current label as being replaced by the outputs of the equivalent label. Otherwise you add the key to the equivalent table, associated with the current label.

This simple algorithm is enough to recursively eliminate common subexpressions. Sometimes the recursive aspect (i.e. noticing that *b* should be replaced by *a*), along with the creation of a common key, causes the technique to be called global value numbering (GVN), but CSE seems a better name to me.

The algorithm as outlined above eliminates expressions that bind values. However not all expressions do that; some are used as control-flow branches. For this reason, Guile also computes a "truthy table" with another flow analysis pass. This table computes a set of which branches have been taken to get to each program point. In the elimination phase, if a branch is reached that is equivalent to a previously taken branch, we consult the truthy table to see which continuation the previous branch may have taken. If it can be proven to have taken just one of the legs, the test is elided and replaced with a direct jump.

A few things to note before moving on. First, the "compute an analysis, then transform the function" sequence is quite common in this sort of problem. It leads to some challenges regarding space for the analysis; [my last article](//wingolog.org/archives/2014/07/01/flow-analysis-in-guile) deals with these in more detail.

Secondly, the rewriting phase assumes that a value that is available may be substituted, and that the result would be a proper CPS term. This isn't always the case; see the discussion at the end of [the article on CSE in Guile 2.0](//wingolog.org/archives/2012/05/14/doing-it-wrong-cse-in-guile) about CPS, SSA, dominators, and scope. In essence, the scope tree doesn't necessarily reflect the dominator tree, so not all transformations you might like to make are syntactically valid. In Guile 2.2's CSE pass, we work around the issue by concurrently rewriting the scope tree to reflect the dominator tree. It's something I am seeing more and more and it gives me some pause as to the suitability of CPS as an intermediate language.

Also, consider the clobbering part of analysis, where e.g. an expression that writes a value to memory has to invalidate previously read values. Currently this is implemented by [traversing all available expressions](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/cse.scm;h=3a03ede83aa4da7d778e1eabf45892db7640bda5;hb=HEAD#l97). This is suboptimal and could be quadratic in the end. A better solution is to compute a dependency graph for expressions, which links together operations on the same regions of memory; see [LLVM's memory dependency analysis](http://llvm.org/docs/doxygen/html/MemoryDependenceAnalysis_8h_source.html) for an idea of how to do this.

Finally, note that this algorithm is global but intraprocedural, meaning that it doesn't propagate values across closure boundaries. It's possible to extend it to be interprocedural, though it's less necessary in the presence of contification.

**scalar replacement via fabricated expressions**

Let's say you get to an expression at label `L`, `(cons a b)`. It binds a result `c`. You determine you haven't seen it before, so you add `(primcall cons a b) → L, c` to your equivalent expressions set. Cool. We won't be able to replace a future instance of `(cons a b)` with `c`, because that doesn't preserve object identity of the newly allocated memory, but it's definitely a cool fact, yo.

What if we add an additional mapping to the table, `(car c) → L, a`? That way any expression at which L is available would replace `(car c)` with `a`, which would be pretty neat. To do so, you would have to [add the `&read` effect to the `cons` call's effects analysis](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/effects-analysis.scm;h=246b22eb6af6c7bb1ced67b7f53541b96b606c71;hb=HEAD#l477), but since the `cons` wasn't really up for elimination anyway it's all good.

Similarly, for `(set-car! c d)` we can add a mapping of `(car c) → d`. Again we have to add the `&read` effect to the `set-car`, but that's OK too because the write invalidated previous reads anyway.

The same sort of transformation holds for [other kinds of memory that Guile knows how to allocate and mutate](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/cse.scm;h=3a03ede83aa4da7d778e1eabf45892db7640bda5;hb=HEAD#l311). Taken together, they form a sort of store-to-load forwarding and scalar replacement that can entirely eliminate certain allocations, and many accesses as well. To actually eliminate the allocations requires a bit more work, but that will be the subject of the next article.

**future work**

So, that's CSE in Guile 2.0. It works pretty well. In the future I think it's probably worth considering an [abstract heap](http://trac.webkit.org/browser/trunk/Source/JavaScriptCore/dfg/DFGAbstractHeap.h#L35)-style analysis of effects; in the end, the precision of CSE is limited to how precisely we can model the effects of expressions.

The trick of using CSE to implement scalar replacement is something I haven't seen elsewhere, though I doubt that it is novel. To fully remove the intermediate allocations needs a couple more tricks, which I will write about in my next nargy dispatch. Until then, happy hacking!

## related articles

- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [cps soup](/archives/2015/07/27/cps-soup)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### One response

1. tomás zerolo says:[26 August 2014 9:15 AM](#c5884d0a5516725be3b291e3eb84f2ac61ea09f6)

   "I know you all have been waiting for the last 27 months for an update, probably forgoing meaningful social interaction in the meantime because what if I posted a followup while you were gone?"

   I even have forgotten eating ;-)

   Now seriously: thoroughly enjoying your posts. Sometimes way over the top of my head, but you possess the gift of putting the right ramps at the right places to make the journey still enjoing. Thanks for sharing that gift!

Comments are closed.
