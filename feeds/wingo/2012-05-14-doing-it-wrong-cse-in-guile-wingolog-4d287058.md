---
title: 'doing it wrong: cse in guile — wingolog'
url: https://wingolog.org/archives/2012/05/14/doing-it-wrong-cse-in-guile
published: "2012-05-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/05/14/doing-it-wrong-cse-in-guile
---

# doing it wrong: cse in guile — wingolog

## [doing it wrong: cse in guile](/archives/2012/05/14/doing-it-wrong-cse-in-guile)

14 May 2012 5:07 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [cse](/tags/cse)
- [optimization](/tags/optimization)
- [peval](/tags/peval)

Greetings, readers! It's been a little while, but not because I haven't been doing anything, or nothing worth writing about. No, the reason I haven't written recently is because the perceived range of my ignorance has been growing faster than the domain of my expertise.

Knowledge may well be something one can dominate, but ignorance must forever be a range, stretching off to a hazy horizon. Climbing the hill of JavaScript implementations has let me see farther out on that plain. I think it's only the existence of the [unknown unknowns](http://en.wikipedia.org/wiki/There_are_known_knowns) that can let one muster up the arrogance to write at all.

But back to domains and dominators later. To begin, I note that there is very little in the way of correct, current, and universal folk wisdom as to how to implement a programming language. Compiler hackers are priests of their languages, yes, but their temples are in reality more or less isolated cults, in which the values of one's Gods may be unknown or abhorrent to those of others. Witness the attention paid to loop optimizations in some languages, compared to garbage collection in others, or closures in still others.

In my ecumenical capacity as abbot of Guile and adjunct deacon of JavaScriptCore, sometimes I'm tempted to mix rites: to sprinkle the holy water of lexical scope optimizations on JS, and, apropos of this article, to exorcise common subexpressions in Scheme.

As one might well imagine, the rites of one cult must be adapted to circumstances. I implemented CSE for Guile, but I don't know if it was actually a win. In this article I'll go into what CSE is, how it works in Guile, why it probably won't survive in its present form.

**cse: common subexpression elimination**

I implemented a source-to-source optimization pass in [Guile](http://gnu.org/s/guile/) that eliminates common subexpressions. It actually does both more and less than that: it propagates predicates and eliminates effect-free statements as well, and these latter optimizations are why I implemented the pass in the first place.

Let me give an example. Let's imagine we implement a binary tree in Guile, using the [records facility](http://www.gnu.org/software/guile/manual/html_node/SRFI_002d9.html).

```
(use-modules (srfi srfi-9))

(define-record-type btree
  (make-btree elt left right)
  btree?
  (elt btree-elt)
  (left btree-left)
  (right btree-right))

(define *btree-null* #f)

(define (btree-cons head tail)
  (if (btree? tail)
      (let ((elt (btree-elt tail)))
        (if (< elt head)
            (make-btree elt
                        (btree-left tail)
                        (btree-cons head (btree-right tail)))
            (make-btree elt
                        (btree-cons head (btree-left tail))
                        (btree-right tail))))
      (make-btree head
                  *btree-null*
                  *btree-null*)))

```

That's enough to illustrate my point, I think. We have the data type, the base case, and a constructor. Of course in Haskell or something with better data types it would be much cleaner, but hey, let's roll with this.

If you look at `btree-cons`, it doesn't seem to be amenable in its current form to classic common subexpression elimination. People don't tend to write duplicated code. You see that I bound the temporary elt instead of typing `(btree-elt btree)` each time, and that was partly because of typing, and partly out of some inner efficiency puritan, telling me I shouldn't write duplicate expressions. (Cult behavior, again!)

But, note that all of these record abstractions will probably inline, instead of calling out to procedures. (They are implemented as [inlinable procedures](http://www.gnu.org/software/guile/manual/html_node/Inlinable-Procedures.html). Yes, it would be better to have cross-module inlining, but we don't, so this is what we do.) In general, syntactic abstraction in Scheme can lead to duplicate code. Apologies in advance for this eyeball-rending torrent, but here's a listing of what `btree-cons` reduces to, after expansion and partial evaluation:

```
(define (btree-cons head tail)
  (if (and (struct? tail)
           (eq? (struct-vtable tail) btree))
      (let ((elt (if (eq? (struct-vtable tail) btree)
                     (struct-ref tail 0)
                     (throw 'wrong-type-arg
                            'btree-elt
                            "Wrong type argument: ~S"
                            (list tail)
                            (list tail)))))
        (if (< elt head)
            (let ((left (if (eq? (struct-vtable tail) btree)
                            (struct-ref tail 1)
                            (throw 'wrong-type-arg
                                   'btree-left
                                   "Wrong type argument: ~S"
                                   (list tail)
                                   (list tail))))
                  (right (btree-cons
                           head
                           (if (eq? (struct-vtable tail) btree)
                               (struct-ref tail 2)
                               (throw 'wrong-type-arg
                                      'btree-right
                                      "Wrong type argument: ~S"
                                      (list tail)
                                      (list tail))))))
              (make-struct/no-tail btree elt left right))
            (let ((left (btree-cons
                          head
                          (if (eq? (struct-vtable tail) btree)
                              (struct-ref tail 1)
                              (throw 'wrong-type-arg
                                     'btree-left
                                     "Wrong type argument: ~S"
                                     (list tail)
                                     (list tail)))))
                  (right (if (eq? (struct-vtable tail) btree)
                             (struct-ref tail 2)
                             (throw 'wrong-type-arg
                                    'btree-right
                                    "Wrong type argument: ~S"
                                    (list tail)
                                    (list tail)))))
              (make-struct/no-tail btree elt left right))))
      (let ((left *btree-null*) (right *btree-null*))
        (make-struct/no-tail btree head left right))))

```

Again, I'm really sorry about that, and it's not just for your eyes: it's also because that's a crapload of code for what should be a simple operation. It's also redundant! There are 6 checks that btree is in fact a btree, when only one is needed semantically. (Note that the null case is not a btree, of course.)

Furthermore, all of the checks in the first arm of the `if` are redundant. The code above is what the optimizer produces -- which is, you know, turrible.

So, I thought, we could run a pass over the source that tries to propagate predicates, and then tries to fold predicates whose boolean value we already know.

And that's what I did. Here's what Guile's optimizer does with the function, including the CSE pass:

```
(define (btree-cons head tail)
  (if (and (struct? tail)
           (eq? (struct-vtable tail) btree))
      (let ((elt (struct-ref tail 0)))
        (if (< elt head)
            (let ((left (struct-ref tail 1))
                  (right (btree-cons head (struct-ref tail 2))))
              (make-struct/no-tail btree elt left right))
            (let ((left (btree-cons head (struct-ref tail 1)))
                  (right (struct-ref tail 2)))
              (make-struct/no-tail btree elt left right))))
      (let ((left *btree-null*) (right *btree-null*))
        (make-struct/no-tail btree head left right))))

```

This is much better. It's quite close to the source program, except the symbolic references like `btree-elt` have been replaced with indexed references. The type check in the predicate of the `if` expression propagated to all the other type checks, causing those nested `if` expressions to fold.

Of course, CSE can also propagate bound lexicals:

```
(let ((y (car x)))
  (car x))
=> (let ((y (car x)))
     y)

```

This is the classic definition of CSE.

**but is it a win?**

I should be quite pleased with the results, except that CSE makes Guile's compiler approximately twice as slow. Granted, in the long run, this should be acceptable: code is usually run many more times than it is compiled. But this is a fairly expensive pass, and yet at the same time it's not as good as it could be.

In order to get to the heart of the matter, I need to explain a little about the implementation. CSE is a post-pass, that runs after [partial evaluation](//wingolog.org/archives/2011/10/11/partial-evaluation-in-guile) (peval). I tried to make it a part of peval, as the two optimizations are synergistic -- oh yes, let's revel in that word -- are you feeling it? -- but it was too complicated in the end. The reason is that in functions like this:

```
(define (explode btree)
  (unless (btree? btree)
    (error "not a btree" btree))
  (values (btree-head btree)
          (btree-left btree)
          (btree-right btree)))

```

Here we have a sequence of two expressions. Since the first one bails out if the predicate is false, we should propagate a true predicate past the first expression. This means that running CSE on an expression returns two values: the rewritten expression, given the predicates already seen; and a new set of predicates that the expression asserts. We should be able to use these new assertions to elide the type checks in the second expression. And indeed, Guile can do this.

Perhaps you can see the size of the problem now. CSE is a pass that runs over all code, building up an ordered set of expressions that were evaluated, and in what context. When it sees a new expression in a test context -- as the predicate in an `if` \-\- it checks to see if the set contains that expression (or its negation) already, in test context, and if so tries to fold the expression to true or false. Already doing this set lookup and traversal is expensive -- at least N log N in the total size of the program, with some quadratic components in the case an expression is found, and also with high constant factors due to the need for custom hash and equality predicates.

The quadratic factor comes in when walking the set to see if the elimination is valid. Consider:

```
(if (car x)
    (if (car x) 10 20)
    30)

```

Here, we should be able to eliminate the second `(car x)`, folding to `(if (car x) 10 30)`. However, in this one:

```
(if (car x)
    (begin
      (y)
      (if (car x) 10 20))
    30)

```

If we don't know what `(y)` does, then we can't fold the second test, because perhaps `(y)` will change the contents of the pair, `x`. The information that allows us to make these decisions is *effects analysis*. For the purposes of Guile's optimizer, `(car x)` has two dependencies and can cause two effects: it depends on the contents of a mutable value, and on the value of a toplevel ( `x`), and can cause the effect of an unbound variable error when resolving the toplevel, or a type error when accessing its car. Two expressions commute if neither depends on effects that the other causes.

I stole the idea of doing a coarse effects analysis, and representing it as bits in a small integer, from V8. Guile's version is here: [effects.scm](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/tree-il/effects.scm). The ordered set is a form of global value numbering. See the CSE pass here: [cse.scm](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/tree-il/cse.scm).

The commute test is fairly cheap, but the set traversal is currently a bit expensive.

**and for what?**

As I have indicated, the pass does do something useful on real programs, as in the binary tree example. But it does not do all it could, and it's difficult to fix that, for a few reasons.

Unlike traditional CSE, Guile's version of it is interprocedural. Instead of operating on just one basic block or one function, it operates across nested functions as well. However, only some dependencies can commute across a function boundary. For example:

```
(lambda (x)
  (if (pair? x)
      (let ((y (car x)))
        (lambda ()
          (and (pair? x) (car x))))))

```

Can the first `pair?` test propagate to the second expression? It can, because `pair?` does not depend on the values of mutable data, or indeed on any effect. If it's true once, it will always be true.

But can we replace the second `(car x)` with `y`? No, because `(car x)` has a dependency on mutable data, and because we don't do escape analysis on the closure, we don't let those dependencies commute across a procedure boundary. (In this case, even if we did escape analysis, we'd have the same conclusion.)

However, not all `lambda` abstractions are closures. Some of them might end up being compiled to labels in the function. Scheme uses syntactically recursive procedures to implement loops, after all. But Guile's CSE does poorly for `lambda` expressions that are actually labels. The reason is that *lexical scope is not a dominator tree*.

MLton hacker Stephen Weeks says it better than I do:

> Thinking of it another way, both CPS and SSA require that variable definitions dominate uses. The difference is that using CPS as an IL requires that all transformations provide a proof of dominance in the form of the nesting, while SSA doesn't. Now, if a CPS transformation doesn't do too much rewriting, then the partial dominance information that it had from the input tree is sufficient for the output tree. Hence tree splicing works fine. However, sometimes it is not sufficient.
>
> As a concrete example, consider common-subexpression elimination. Suppose we have a common subexpression `x = e` that dominates an expression `y = e` in a function. In CPS, if `y = e` happens to be within the scope of `x = e`, then we are fine and can rewrite it to `y = x`. If however, `y = e` is not within the scope of `x`, then either we have to do massive tree rewriting (essentially making the syntax tree closer to the dominator tree) or skip the optimization. Another way out is to simply use the syntax tree as an approximation to the dominator tree for common-subexpression elimination, but then you miss some optimization opportunities. On the other hand, with SSA, you simply compute the dominator tree, and can always replace `y = e` with `y = x`, without having to worry about providing a proof in the output that `x` dominates `y`. (i.e. without putting `y` in the scope of `x`)
>
> [\[MLton-devel\] CPS vs SSA](http://mlton.org/pipermail/mlton/2003-January/023054.html)

See my [article on SSA and CPS](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers) for more context.

So that's one large source of lost CSE opportunities, especially in loops.

Another large source of lost opportunities is that the Tree-IL language, which is basically a macro-expanded Scheme, has the same property that Scheme does, that the order of evaluation of operands is unspecified.

Consider the base-case clause of my `btree-cons` above:

```
(let ((left *btree-null*) (right *btree-null*))
  (make-struct/no-tail btree head left right))

```

Now, `*btree-null*` is a toplevel lookup, that might have an unbound-variable effect. We should be able to eliminate one of them, though. Why doesn't the CSE pass do it? Because in Tree-IL, the order of evaluation of the right-hand-sides of left and right is unspecified. This gives Guile some useful freedoms, but little information for CSE.

This is an instance of a more general problem, that Tree-IL might be too high-level to be useful for CSE. For example, at runtime, not all lexical references are the same -- some are local, and some reference free variables. For mutated free variables, the variable is itself in a box, so to reference it you would load the box into a local and then dereference the box. CSE should allow you to eliminate duplicate loads of the box, even in the case that it can't eliminate duplicate references into the box.

**conclusion**

It is nice to be able to eliminate the duplicate checks, but not at any price. Currently the bootstrapping time cost is a bit irritating. I have other ideas on how to fix that, but ultimately we probably need to re-implement CSE at some lower level. More on that in a future post. Until then, happy hacking.

## related articles

- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [cps soup](/archives/2015/07/27/cps-soup)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)

### 11 responses

1. [Paul Khuong](http://www.pvk.ca) says:[15 May 2012 10:13 AM](#1fdebb3435a5b5dafd9233ae7637264c772e4db4)

   If you only wish to elide redundant type checks, there are simpler (to my mind) ways, simply by flowing the information about predicates (on assignment-free bindings) into conditional branches. It seems easier to me to flow type information about bindings \[e.g. if X isn't assigned to, it is always btree? in the true branch of (if (branch? x) ...)\] than to try and recognize equivalent code sequences... and, unlike general CSE, optimising conditional branches away is alway a win.

2. [wingo](http://wingolog.org/) says:[15 May 2012 10:40 AM](#6263328e178626db1fd6a19e3079ccdc6301c4c0)

   Hey Paul, thanks for stopping by.

   One of the problems is that after expansion, there is no "btree?" in the source -- it expands out to `(and (struct? x) (eq? (struct-vtable x) btree))`.

   Given that, it was just as easy to match arbitrary expressions as to special-case some information. In addition, one problem here is that the vtable referenced in the sub-check, `btree` in this case, might be a mutable toplevel value. In that case it's useful to be able to treat it as an arbitrary expression, already having exhibited the &type-check effects (see the effects.scm).

   However, you are certainly right that having predicate information is more useful overall, and it might well be the right thing, going forward. The naive approach Guile's optimizer uses won't help very much at eliding checks for related accessors; e.g. it might have already proved `(pair? x)`, but without special checks, it don't know that `(car x)` is valid.

   I'm sure the static typing people are laughing at us, at this point ;-)

3. gasche says:[15 May 2012 11:45 AM](#e4e81c9b989f1d547ddffe72576b6f684622e151)

   \> I'm sure the static typing people are laughing at us, at this point ;-)

   Typed Racket work on predicate typing is quite serious and is, I suppose, state of the art if you want to do this kind of things. At least I haven't seen anybody laughing at it.

   The static typing people could say that algebraic datatypes plus pattern matching is a better way to write such code, and is also easier to type-check (because case branches naturally give you typing information, contrary to if/then/else test that needs more complex analyses to statically analyze). But if you want to type-check or optimize idiomatic Lisp/Scheme code, well, we understand that you have to do this.

4. [Paul Khuong](http://www.pvk.ca) says:[15 May 2012 2:27 PM](#0d8971f1c9e6202317fe262a00ede0f0efd788bd)

   \> One of the problems is that after expansion, there is no "btree?" in the source -- it expands out to (and (struct? x) (eq? (struct-vtable x) btree)).

   Ah, that eq? check would indeed be a pain. Is there no easy way to run some analyses before (and after) inlining?

   \> But if you want to type-check or optimize idiomatic Lisp/Scheme code, well, we understand that you have to do this.

   Right. I find that's pretty much the only interesting bit in SBCL and CMUCL.

5. [Smarter](http://smarter.free.fr) says:[15 May 2012 3:30 PM](#17bc95af7e833128d327552d236ef54b4b00e451)

   Nice post! It looks like you use both btree-elt and btree-head to mean the same thing.

6. [wingo](http://wingolog.org) says:[15 May 2012 3:57 PM](#7d6f9b9f2295cf37a72d363b065dfd03b893af00)

   Thanks for the note, Smarter; fixed (hopefully).

7. [Greg Benison](http://gcbenison.wordpress.com) says:[15 May 2012 11:18 PM](#c134c59a50518924d23cd887d165b4c9fafbb7c7)

   How come nobody ever talks about the unknown knowns?

8. [Neel Krishnaswami](http://semantic-domain.blogspot.com) says:[16 May 2012 6:16 AM](#014d72e2a416c33fb271db1eec22c7f633735ed9)

   *I'm sure the static typing people are laughing at us, at this point ;-)*

   No, we're not. If we ever want high-level languages to seriously compete with low-level languages, we need to do bulk transformations like fusion (eg, turn `(map f (map g xs))` into `(map (compose f g) xs)`.

   But to get those transformations to fire reliably, we REALLY need high-quality flow analysis for higher-order languages. Without it, these kinds of rewritings are very, very brittle. See ghc Haskell's rules mechanism for an example of the potential power and practical brittleness of these kinds of rewritings.

   The difference between Scheme and Haskell is, at this point, just a matter of degree -- in both cases we need to flow information that's not in the types through the program. It's just that Scheme has one type and Haskell has lots of them.

9. [Krzysztof Stachowiak](http://stah.ovh.org) says:[17 May 2012 10:17 AM](#18d8f7113574e8cfad3c3287be51903fe0163c15)

   \> How come nobody ever talks about the unknown knowns?

   I think the data mining people do it all the time.

10. anon says:[20 May 2012 6:24 PM](#fdc7538561e5946c8e4a25fb39ff96c2d9132cf2)

    You know about "A Datatype Macro for Scheme" ( http://www.cs.indiana.edu/~jsobel/Recycling/datatype.html ), right?

11. agumonkey says:[7 April 2014 9:46 AM](#3802aeed73bdd4855ce46d5728fccd3f32a39a59)

    undeaded link for "A Datatype Macro for Scheme" http://web.archive.org/web/20120930032951/http://www.cs.indiana.edu/~jsobel/Recycling/datatype.html

Comments are closed.
