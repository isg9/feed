---
title: loop optimizations in guile — wingolog
url: https://wingolog.org/archives/2015/07/28/loop-optimizations-in-guile
published: "2015-07-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/07/28/loop-optimizations-in-guile
---

# loop optimizations in guile — wingolog

## [loop optimizations in guile](/archives/2015/07/28/loop-optimizations-in-guile)

28 July 2015 8:10 AM

- [licm](/tags/licm)
- [loop peeling](/tags/loop%20peeling)
- [code motion](/tags/code%20motion)
- [loop inversion](/tags/loop%20inversion)
- [loop identification](/tags/loop%20identification)
- [guile](/tags/guile)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [code hoisting](/tags/code%20hoisting)
- [effects analysis](/tags/effects%20analysis)

Sup peeps. So, after the [slog to update Guile's intermediate language](https://wingolog.org/archives/2015/07/27/cps-soup), I wanted to land some new optimizations before moving on to the next thing. For years I've been meaning to do some loop optimizations, and I was finally able to land a few of them.

**loop peeling**

For a long time I have wanted to do "loop peeling". Loop peeling means peeling off the first iteration of a loop. If you have a source program that looks like this:

```
while foo:
  bar()
  baz()

```

Loop peeling turns it into this:

```
if foo:
  bar()
  baz()
  while foo:
    bar()
    baz()

```

You wouldn't think that this is actually an optimization, would you? Well on its own, it's not. But if you combine it with common subexpression elimination, then it means that the loop body is now dominated by all effects and all loop-invariant expressions that must be evaluated for the expression to loop.

In dynamic languages, this is most useful when one source expression expands to a number of low-level steps. So for example if your language runtime implements top-level variable references in three parts, one where it gets a reference to a mutable box, then it checks if the box has a value, and and the third where it unboxes it, then we would have:

```
if foo:
  bar_location = lookup("bar")
  bar_value = dereference(bar_location)
  if bar_value is null: throw NotFound("bar")
  call(bar_value)

  baz_location = lookup("baz")
  baz_value = dereference(baz_location)
  if baz_value is null: throw NotFound("baz")
  call(baz_value)

  while foo:
    bar_value = dereference(bar_location)
    call(bar_value)

    baz_value = dereference(baz_location)
    call(baz_value)

```

The result is that we have hoisted the lookups and null checks out of the loop (if a box can never transition from full back to empty). It's a really powerful transformation that can even hoist things that traditional loop-invariant code motion can't, but more on that later.

Now, the problem with loop peeling is that usually values will escape your loop. For example:

```
while foo:
  x = qux()
  if x then return x
...

```

In this little example, there is a value `x`, and the `return x` statement is actually not in the loop. It's syntactically in the loop, but the underlying representation that the compiler uses looks more like this:

```
function qux(k):
  label loop_header():
    fetch(foo) -gt; loop_test
  label loop_test(foo_value):
    if foo_value then -> exit else -> body
  label body():
    fetch(x) -gt; have_x
  label have_x(x_value):
    if x_value then -> return_x else -> loop_header
  label return_x():
    values(x) -> k
  label exit():
    ...

```

This is the ["CPS soup"](https://wingolog.org/archives/2015/07/27/cps-soup) I described in my last post. Only the bold parts are in the loop; notably, the `return` is outside the loop. Point being, if we peel off the first iteration, then there are two possible values for `x` that we would return:

```
if foo:
  x1 = qux()
  if x1 then return x1
  while foo:
    x2 = qux()
    if x2 then return x2
  ...

```

I have them marked as `x1` and `x2`. But I've also duplicated the `return x` terms, which is not what we want. We want to peel off the first iteration, which will cause code growth equal to the size of the loop body, but we don't want to have to duplicate everything that's after the loop. What we have to do is re-introduce a join point that defines `x`:

```
if foo:
  x1 = qux()
  if x1 then join(x1)
  while foo:
    x2 = qux()
    if x2 then join(x2)
  ...
label join(x)
  return x

```

Here I'm playing fast and loose with notation because the real terms are too gnarly. What I'm trying to get across is that for each value that flows out of a loop, you need a join point. That's fine, it's a bit more involved, but what if your loop exits to two different points, but one value is live in both of them? A value can only be defined in one place, in CPS or SSA. You could re-place a whole tree of phi variables, in SSA parlance, with join blocks and such, but it's just too hard.

However we can still get the benefits of peeling in most cases if we restrict ourselves to loops that exit to only one continuation. In that case the live variable set is the intersection of all variables defined in the loop that are live at the exit points. Easy enough, and that's what we have in Guile now. Peeling causes some code growth but the loops are smaller so it should still be a win. [Check out the source](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/peel-loops.scm;h=a1b04a45bca24035a8f0e329e2a13982495fe1b9;hb=HEAD), if that's your thing.

**loop-invariant code motion**

Usually when people are interested in moving code out of loops they talk about loop-invariant code motion, or LICM. Contrary to what you might think, LICM is complementary to peeling: some things that peeling+CSE can hoist are not hoistable by LICM, and vice versa.

Unlike peeling, LICM does not cause code growth. Instead, for each expression in a loop, LICM tries to hoist it out of the loop if it can. An expression can be hoisted if all of these conditions are true:

1. It doesn't cause the creation of an observably new object. In Scheme, [the definition of "observable" is quite subtle](http://wingolog.org/archives/2013/11/02/scheme-quiz-time), so in practice in Guile we don't hoist expressions that can cause any allocation. We could use alias analysis to improve this.

2. The expression cannot throw an exception, *or* the expression is always evaluated for every loop iteration.

3. The expression makes no writes to memory, or if it writes to memory, other expressions in the loop cannot possibly read from that memory. We use [effects analysis](http://wingolog.org/archives/2014/05/18/effects-analysis-in-guile) for this.

4. The expression makes no reads from memory, or if it reads from memory, no other expression in the loop can clobber those reads. Again, effects analysis.

5. The expression uses only loop-invariant variables.

This definition is inductive, so once an expression is hoisted, the values it defines are then considered loop-invariant, so you might be able to hoist a whole chain of values.

Compared to loop peeling, this has the gnarly aspect of having to explicitly reason about loop invariance and manually move code, which is a pain. (Really LICM would be better named "artisanal code motion".) However it causes no code growth, which is a plus, though like peeling it can increase register pressure. But the big difference is that LICM can hoist effect-free expressions that aren't always executed. Consider:

```
while foo:
  x = qux() ? "hi" : "ho"

```

Here for some reason it could be faster to cache "hi" or "ho" in registers, which is what LICM allows:

```
hi, ho = "hi", "ho"
while foo:
  x = qux() ? hi : ho

```

On the other hand, LICM alone can't hoist the `if baz is null` checks in this example from above:

```
while foo:
  bar()
  baz()

```

The issue is that the call to bar() might not return, so the error that might be thrown if `baz` is null shouldn't be observed until `bar` is called. In general we can't hoist anything that might throw an exception past some non-hoisted code that might throw an exception. This specific situation happens in Guile but there are similar ones in any language, I think.

More formally, LICM will hoist effectful but loop-invariant expressions that postdominate the loop header, whereas peeling hoists those expressions that dominate all back-edges. I think? We'll go with that. Again, [the source](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/licm.scm;h=3b343a66bd8ed4a591a9e97edbf1179a4d3a78a8;hb=HEAD).

**loop inversion**

Loop inversion is a little hack to improve code generation, and again it's a little counterintuitive. If you have this loop:

```
while n < x:
  n++

```

Loop inversion turns it into:

```
if n < x:
  do
    n++
  while n < x

```

The goal is that instead of generating code that looks like this:

```
header:
  test n, x;
  branch-if-greater-than-or-equal done;
  x = x + 1
  goto header
done:

```

You make something that looks like this:

```
  test n, x;
  branch-if-greater-than-or-equal done;
header:
  x = x + 1
  test n, x;
  branch-if-less-than header;
done:

```

The upshot is that the loop body now contains one branch instead of two. It's mostly helpful for tight loops.

It turns out that you can express this transformation on CPS (or SSA, or whatever), but that like loop peeling the extra branch introduces an extra join point in your program. If your loop exits to more than one label, then we have the same problems as loop peeling. For this reason Guile restricts loop inversion (which it calls "loop rotation" at the moment; I should probably fix that) to loops with only one exit continuation.

Loop inversion has some other caveats, but probably the biggest one is that in Guile it doesn't actually guarantee that each back-edge is a conditional branch. The reason is that usually a loop has some associated loop variables, and it could be that you need to reshuffle those variables when you jump back to the top. Mostly Guile's compiler manages to avoid shuffling, allowing inversion to have the right effect, but it's not guaranteed. Fixing this is not straightforward, since the shuffling of values is associated with the predecessor of the loop header and not the loop header itself. If instead we reshuffled before the header, that might work, but each back-edge might have a different shuffling to make... anyway. In practice inversion seems to work out fine; I haven't yet seen a case where it doesn't work. [Source code here](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/rotate-loops.scm;h=0fab94f1dac4c660902fae08282bfea4c31610a5;hb=HEAD).

**loop identification**

One final note: what is a loop anyway? Turns out this is a somewhat hard problem, especially once you start trying to identify nested loops. Guile currently does the simple thing and just computes [strongly-connected components](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/utils.scm;h=fcbda9e76b82b1f607a39f4c00fc17f323ea3fda;hb=HEAD#l361) in a function's flow-graph, and says that a loop is a non-trivial SCC with a single predecessor. That won't tease apart loop nests but oh wells! I spent a lot of time last year or maybe two years ago with that "Loop identification via D-J graphs" paper but in the end simple is best, at least for making incremental steps.

Okeysmokes, until next time, loop on!

## related articles

- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [cps soup](/archives/2015/07/27/cps-soup)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [doing it wrong: cse in guile](/archives/2012/05/14/doing-it-wrong-cse-in-guile)

### One response

1. [Peter Goodman](http://www.petergoodman.me) says:[28 July 2015 2:01 PM](#bc95163982e411a69cf8908efc54bc177614ccbc)

   Loop inversion seems like a nice way of handling this. I wonder if an alternative is a CPS-equivalent version of single-static use form (see PhD thesis of John Plevyak). With SSA, you have the PHI node as a join point. SSU is an extension where you have a PSI node where, if a value is used on two sides of a branch, then you introduce PSI(x1, x2) = x0 before the branch/loop header, and then use x1 or x2 on each side, and end with x3 = PHI(x1, x2).

Comments are closed.
