---
title: effects analysis in guile — wingolog
url: https://wingolog.org/archives/2014/05/18/effects-analysis-in-guile
published: "2014-05-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/05/18/effects-analysis-in-guile
---

# effects analysis in guile — wingolog

## [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)

18 May 2014 7:19 PM

- [guile](/tags/guile)
- [compilers](/tags/compilers)
- [optimization](/tags/optimization)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [anf](/tags/anf)
- [cse](/tags/cse)
- [v8](/tags/v8)
- [igalia](/tags/igalia)

OK kids, so I had a bit of time recently. I've been hacking on Guile's new CPS-based compiler, which should appear in a stable release in a few months. I have a few things to write about, but today's article is on effects analysis.

**what it is, yo**

The job of the optimization phase of a compiler is to remove, replace and reorder the sub-expressions of a program. The optimizer recognizes ways in which the program can be better, and then needs to check if those transformations are valid. A transformation is valid if a program exhibits the same behavior both before and after the transformation.

Effects analysis is a proof technique that builds a conservative model of how expressions can affect each other. It can be used to prove the validity of some transformations. For example, if we determine that an expression A reads the first field in a vector, and expression B sets the second field in a vector, then we can use effects analysis to show that the expressions don't affect each others' value and can be freely reordered, for example to hoist either one out of a loop.

In effects analysis, we model the program state coarsely and conservatively with a limited set of categories. The precise set of categories depends on the domain. In graphics, for example, you might have a state bit for the current coordinate system, the current color, the current fill mode, etc. In general-purpose compilation, the effects are mostly about memory and (sometimes) exceptions.

In Guile we model four kinds of effects: type checks (T), allocations (A), reads (R), and writes (W). Each of these effects is allocated to a bit. An expression can have any or none of these effects.

![](//wingolog.org/pub/effects-analysis.png)

For the purposes of Guile, a "type check" is the possibility that this expression will throw an exception if its arguments are somehow out of range. For example, `cons` will never throw an exception, except in out-of-memory situations, but `+` may throw an exception if its arguments aren't numbers. However if an expression is dominated by a copy of itself -- that is, if we see that `(+ a b)` (which may throw an exception) is dominated by `(+ a b)`, perhaps after CSE -- then we can determine that only the first will exhibit the type-check effects, and that the second will not.

Allocation indicates that an expression allocates a fresh object. In Scheme (and many other languages), each allocated object has its own identity: `(eq? (cons 1 2) (cons 1 2))` must be false. Note that this restriction does not apply to constant literals, in Scheme; `(eq? '(1 . 2) '(1 . 2))` may or may not be true. In Guile both results are possible. They are the same object when compiled (and thus deduplicated), but different when interpreted; the objects are just the ones returned from \`read' and this are different. Anyway we use this allocation bit to indicate that an expression allocates a fresh object with a fresh identity.

The remaining effect kinds are "read" and "write", which indicate reads or writes to memory. So there are just 4 kinds of effects.

Allocation, read, and write effects are associated at run-time with particular memory locations. At compile-time we cannot in general know which of these locations are distinct, and which are actually the same. To simplify the problem, we simply record the type of the object, knowing that (say) a pair and a vector will never be at the same location. We also record the field in the object in the case of objects with multiple fields. There are special catch-all values to indicate "all memory kinds", when we really don't know what an expression will do (which is the case for all expression kinds without specific support in the effect analyzer), and for "all fields" when we don't know which field we are accessing.

One example of the use of this analysis is in common subexpression elimination (CSE). If at a program point P we have a set of available expressions A, then to determine which members of A are still available after the expression at P, you subtract members of A that are clobbered by P. Computation of A at each P plus value numbering is most of CSE; but more on that in some later dispatch. Anyway here's the definition of `effects-clobber?`.

```
(define (effect-clobbers? a b)
  "Return true if A clobbers B.  This is the case
if A might be a write, and B might be a read or a
write to the same location as A."
  (define (locations-same?)
    (let ((a (ash a (- &effect-kind-bits)))
          (b (ash b (- &effect-kind-bits))))
      (or (eqv? &unknown-memory-kinds
                (logand a &memory-kind-mask))
          (eqv? &unknown-memory-kinds
               (logand b &memory-kind-mask))
          (and (eqv? (logand a &memory-kind-mask)
                     (logand b &memory-kind-mask))
               ;; A negative field indicates "the
               ;; whole object".  Non-negative fields
               ;; indicate only part of the object.
               (or (< a 0) (< b 0) (= a b))))))
  (and (not (zero? (logand a &write)))
       (not (zero? (logand b (logior &read &write))))
       (locations-same?)))

```

This analysis is simple, small, and fast. It's also coarse and imprecise -- if you are reading from and writing to two vectors at once, you're almost sure to miss some optimization opportunities as accesses to all vectors are conflated into one bit. Oh well. If you get into this situation, you'll know it, and be able to invest a bit more time into alias analysis; there's lots of literature out there. A simple extension would be to have alias analysis create another mapping from expression to equivalence class, and to use those equivalence classes in the `same-location?` check above.

Of course this assumes that expressions just access one object. This is the case for Guile's CPS intermediate language, because [in CPS, as in SSA or ANF, expressions don't have subexpressions](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers).

This contrasts with direct-style intermediate languages, in which expressions may have nested subexpressions. If you are doing effects analysis on such a language, it's probably more appropriate to allocate a bit to each kind of effect on each kind of object, so that you can usefully union effects for a tree of expressions. Since you don't have to do this for CPS, we can allocate a fixed bit-budget towards more precision as to which field of an object is being accessed. The inability to be precise as to which field was being accessed due to the direct-style IL was [one of the problems in Guile's old CSE pass](//wingolog.org/archives/2012/05/14/doing-it-wrong-cse-in-guile).

Finally, a note about type checks. Guile includes type checks as part of the effects analysis for two reasons. The first is the obvious case of asking whether an expression is effect-free or not, which can lead to some optimizations in other parts of the compiler. The other is to express the potential for elision of duplicate expressions, if one dominates the other. But it's also possible to remove type checks in more cases than that: one can run a type inference pass to remove type-check effects if we can prove that the arguments to an expression are in range. Obviously this is more profitable for dynamically-typed languages, but the same considerations apply to any language with sum types.

Guile's effects analysis pass is [here](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/effects-analysis.scm;hb=HEAD). V8 seems to have two effects analysis passes; one is in [effects.h](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/effects.h) and [typing.cc](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/typing.cc), and operates over the direct-style AST, and the other is in the value numbering pass ( [hydrogen-instructions.h](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/hydrogen-instructions.h) and [hydrogen-gvn.h](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/hydrogen-gvn.h); search for GVNFlag).

More on how effects analysis is used in Guile in a future missive. Until then, happy hacking.

## related articles

- [static single assignment for functional programmers](/archives/2011/07/12/static-single-assignment-for-functional-programmers)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [cps soup](/archives/2015/07/27/cps-soup)
- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)
- [doing it wrong: cse in guile](/archives/2012/05/14/doing-it-wrong-cse-in-guile)

### 3 responses

1. [John Cowan](http://www.ccil.org/~cowan) says:[19 May 2014 8:51 PM](#ec2208ddb03d3dd7a4a3550bc9f29f2b7987deca)

   This, of course, is exactly how effects were treated in the primordial CPS compiler Rabbit. The more it changes, the samer it gets.

2. tomás zerolo says:[12 June 2014 11:33 AM](#e0c16b4c7415b286f263ded1507273ea7549f234)

   Hi, Andy

   again an unspecific comment, but I've got to let you know that I enjoy thoroughly those Guile articles. Not only are you doing an awesome job, but you are making it awesomely accessible with your write-ups.

   Thanks!

3. [Nala Ginrut](http://nalaginrut.com) says:[3 July 2014 3:42 AM](#936fec0204cd86151c8430ece644a0411eab22d7)

   It's a nice introduction for effects analysis in Guile. I've learned much from it. I didn't know the type checks in Guile is part of the effects analysis.

   Thanks!

Comments are closed.
