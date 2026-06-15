---
title: whiffle, a purpose-built scheme — wingolog
url: https://wingolog.org/archives/2023/11/14/whiffle-a-purpose-built-scheme
published: "2023-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/11/14/whiffle-a-purpose-built-scheme
---

# whiffle, a purpose-built scheme — wingolog

## [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)

14 November 2023 10:10 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [whiffle](/tags/whiffle)
- [apologies](/tags/apologies)
- [cse](/tags/cse)
- [peval](/tags/peval)
- [cp0](/tags/cp0)
- [baseline](/tags/baseline)
- [safepoints](/tags/safepoints)

Yesterday I [promised an
apology](https://wingolog.org/archives/2023/11/13/i-accidentally-a-scheme) but didn’t actually get past the admission of guilt. Today the defendant takes the stand, in the hope that an awkward cross-examination will persuade the jury to take pity on a poor misguided soul.

Which is to say, let’s talk about [Whiffle](https://github.com/wingo/whiffle): what it actually is, what it is doing for me, and why on earth it is that \[I tell myself that\] writing a new programming language implementation is somehow preferable than re-using an existing one.

### graphic designgarbage collection is my passion

Whiffle is purpose-built to test the [Whippet](https://github.com/wingo/whippet) garbage collection library.

Whiffle lets me create Whippet test cases in C, without actually writing C. C is fine and all, but the problem with it and garbage collection is that you have to track all stack roots manually, and this is an error-prone process. Generating C means that I can more easily ensure that each stack root is visitable by the GC, which lets me make test cases with more confidence; if there is a bug, it is probably not because of an untraced root.

Also, Whippet is mostly meant for programming language runtimes, not for direct use by application authors. In this use-case, probably you can use less “active” mechanisms for ensuring root traceability: instead of eagerly recording live values in some kind of [handlescope](https://github.com/v8/v8/blob/main/src/handles/handles.h), you can keep a [side
table](https://github.com/v8/v8/blob/7fdc2728f2114b083ebbd24710a32fe4574c0c57/src/codegen/safepoint-table.h#L201) that is only consulted as needed during garbage collection pauses. In particular since Scheme uses the [stack as a data
structure](https://wingolog.org/archives/2014/03/17/stack-overflow), I was worried that using handle scopes would somehow distort the performance characteristics of the benchmarks.

Whiffle is not, however, a high-performance Scheme compiler. It is not for number-crunching, for example: garbage collectors don’t care about that, so let’s not. Also, Whiffle should not go to any effort to remove allocations ( [sroa / gvn /
cse](https://wingolog.org/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)); creating nodes in the heap is the purpose of the test case, and eliding them via compiler heroics doesn’t help us test the GC.

I settled on a [baseline-style](https://wingolog.org/archives/2020/06/03/a-baseline-compiler-for-guile) compiler, in which I re-use the Scheme front-end from [Guile](https://gnu.org/s/guile) to expand macros and create an [abstract syntax
tree](https://www.gnu.org/software/guile/manual/html_node/Tree_002dIL.html). I do run some optimizations on that AST; in the spirit of [the macro
writer’s bill of rights](https://www.youtube.com/watch?v=LIEX3tUliHw), it does make sense to provide some basic reductions. (These reductions can be surprising, but I am used to the [Guile’s flavor of cp0
(peval)](https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile), and this project is mostly for me, so I thought it was risk-free; I was [almost right!](https://mastodon.social/@wingo/111093597389556069)).

Anyway the result is that Whiffle uses an explicit stack. A [safepoint](https://wingolog.org/archives/2023/10/16/on-safepoints) for a thread simply records its stack pointer: everything between the stack base and the stack pointer is live. I do have a lingering doubt about the representativity of this compilation strategy; would a conclusion drawn from Whippet apply to Guile, which uses a different stack allocation strategy? I think probably so but it’s an unknown.

### what’s not to love

Whiffle also has a number of design goals that are better formulated in the negative. I mentioned compiler heroics as one thing to avoid, and in general the desire for a well-understood correspondence between source code and run-time behavior has a number of other corrolaries: Whiffle is a pure ahead-of-time (AOT) compiler, as just-in-time (JIT) compilation adds noise. Worse, speculative JIT would add unpredictability, which while good on the whole would be anathema to understanding an isolated piece of a system like the GC.

Whiffle also should produce stand-alone C files, without a thick run-time. I need to be able to understand and reason about the residual C programs, and depending on third-party libraries would hinder this goal.

Oddly enough, users are also an anti-goal: as a compiler that only exists to test a specific GC library, there is no sense in spending too much time making Whiffle nicer for other humans, humans whose goal is surely not just to test Whippet. Whiffle is an interesting object, but is not meant for actual use or users.

### corners: cut

Another anti-goal is completeness with regards to any specific language standard: the point is to test a GC, not to make a useful Scheme. Therefore Whippet gets by just fine without flonums, fractions, continuations (delimited or otherwise), multiple return values, ports, or indeed any library support at all. All of that just doesn’t matter for testing a GC.

That said, it has been useful to be able to import standard Scheme garbage collection benchmarks, such as [earley](https://github.com/wingo/whiffle/blob/main/examples/earley.scm) or [nboyer](https://github.com/wingo/whiffle/blob/main/examples/nboyer.scm). These have required very few modifications to run in Whippet, mostly related to Whippet’s test harness that wants to spawn multiple threads.

### and so?

I think this evening we have elaborated a bit more about the “how”, complementing yesterday’s note about the “what”. Tomorrow (?) I’ll see if I can dig in more to the “why”: what questions does Whiffle let me ask of Whippet, and how good of a job does it do at getting those answers? Until then, may all your roots be traced, and happy hacking.

## related articles

- [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [whippet progress update: feature-complete!](/archives/2024/09/18/whippet-progress-update-feature-complete)
- [on safepoints](/archives/2023/10/16/on-safepoints)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)

Comments are closed.
