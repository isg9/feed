---
title: cps in hoot — wingolog
url: https://wingolog.org/archives/2024/05/27/cps-in-hoot
published: "2024-05-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/05/27/cps-in-hoot
---

# cps in hoot — wingolog

## [cps in hoot](/archives/2024/05/27/cps-in-hoot)

27 May 2024 12:36 PM

- [guile](/tags/guile)
- [hoot](/tags/hoot)
- [scheme](/tags/scheme)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [wasm](/tags/wasm)
- [webassembly](/tags/webassembly)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [spritely](/tags/spritely)

Good morning good morning! Today I have another article on the [Hoot
Scheme-to-Wasm compiler](https://spritely.institute/hoot), this time on Hoot’s use of the continuation-passing-style (CPS) transformation.

### calls calls calls

So, just a bit of context to start out: Hoot is a Guile, Guile is a Scheme, Scheme is a Lisp, one with [“proper tail
calls”](https://www.cs.tufts.edu/~nr/cs257/archive/will-clinger/proper-tail-space.pdf): function calls are either in tail position, syntactically, in which case they are *tail calls*, or they are not in tail position, in which they are *non-tail calls*. A non-tail call suspends the calling function, putting the rest of it (the *continuation*) on some sort of stack, and will resume when the callee returns. Because non-tail calls push their continuation on a stack, we can call them *push calls*.

```
(define (f)
  ;; A push call to g, binding its first return value.
  (define x (g))
  ;; A tail call to h.
  (h x))

```

Usually the problem in implementing Scheme on other language run-times comes in tail calls, but WebAssembly supports them natively (except on JSC / Safari; should be coming at some point though). Hoot’s problem is the reverse: how to implement push calls?

The issue might seem trivial but it is not. Let me illustrate briefly by describing what Guile does natively (not compiled to WebAssembly). Firstly, note that I am discussing *residual* push calls, by which I mean to say that the optimizer might remove a push call in the source program via inlining: we are looking at those push calls that survive the optimizer. Secondly, note that native Guile [manages its own
stack](https://wingolog.org/archives/2014/03/17/stack-overflow) instead of using the stack given to it by the OS; this allows for push-call recursion without arbitrary limits. It also lets Guile [capture stack
slices and rewind
them](https://wingolog.org/archives/2010/02/26/guile-and-delimited-continuations), which is the fundamental building block we use to implement exception handling, [Fibers](https://wingolog.org/archives/2017/06/27/growing-fibers) and other forms of lightweight concurrency.

The straightforward [function
call](https://webassembly.github.io/gc/core/exec/instructions.html#function-calls) will have an artificially limited total recursion depth in most WebAssembly implementations, meaning that many idiomatic uses of Guile will throw exceptions. Unpleasant, but perhaps we could stomach this tradeoff. The greater challenge is how to slice the stack. That I am aware of, there are three possible implementation strategies.

#### generic slicing

One possibility is that the platform provides a generic, powerful stack-capture primitive, which is what Guile does. The good news is that one day, the [WebAssembly stack-switching
proposal](https://github.com/WebAssembly/stack-switching/blob/main/proposals/continuations/Explainer.md) should provide this too. And in the meantime, the so-called [JS Promise
Integration (JSPI)](https://v8.dev/blog/jspi) proposal gets close: if you enter Wasm from JS via a function marked as async, and you call out to JavaScript to a function marked as async (i.e. returning a promise), then on that nested Wasm-to-JS call, the engine will suspend the continuation and resume it only when the returned promise settles (i.e. completes with a value or an exception). Each entry from JS to Wasm via an async function allocates a fresh stack, so I understand you can have multiple pending promises, and thus multiple wasm coroutines in progress. It gets a little gnarly if you want to control when you wait, for example if you might want to wait on multiple promises; in that case you might not actually mark promise-returning functions as async, and instead import an async-marked `async function waitFor(p) { return await p}` or so, allowing you to use `Promise.race` and friends. The main problem though is that JSPI is only for JavaScript. Also, its stack sizes are even smaller than the the default stack size.

#### instrumented slicing

So much for generic solutions. There is another option, to still use push calls from the target machine (WebAssembly), but to transform each function to allow it to suspend and resume. This is what I think of as [Joe Marshall’s stack
trick](https://funcall.blogspot.com/2008/09/in-from-generalized-stack-inspection.html) (also see [§4.2 of the associated
paper](https://cs.brown.edu/~sk/Publications/Papers/Published/pcmkf-cont-from-gen-stack-insp/)). The idea is that although there is no primitive to read the whole stack, each frame can access its own state. If you insert a `try`/ `catch` around each push call, the catch handler can access local state for activations of that function. You can slice a stack by throwing a `SaveContinuation` exception, in which each frame’s catch handler saves its state and re-throws. And if we want to avoid exceptions, we can use checked returns as [Asyncify](https://kripken.github.io/blog/wasm/2019/07/16/asyncify.html) does.

I never understood, though, how you resume a frame. The Generalized Stack Inspection paper would seem to indicate that you need the transformation to introduce a function to run “the rest of the frame” at each push call, which becomes the `Invoke` virtual method on the reified frame object. To avoid code duplication you would have to make normal execution flow run these `Invoke` snippets as well, and that might undo much of the advantages. I understand the implementation that Joe Marshall was working on was an interpreter, though, which bounds the number of sites needing such a transformation.

#### cps transformation

The third option is a continuation-passing-style transformation. A CPS transform results in a program whose procedures “return” by tail-calling their “continuations”, which themselves are procedures. Taking our previous example, a naïve CPS transformation would reify the following program:

```
(define (f' k)
  (g' (lambda (x) (h' k x))))

```

Here `f'` (“f-prime”) receives its continuation as an argument. We call `g'`, for whose continuation argument we pass a closure. That closure is the return continuation of `g`, binding a name to its result, and then tail-calls `h` with respect to `f`. We know their continuations are the same because it is the same binding, `k`.

Unfortunately we can’t really slice abitrary ranges of a stack with the naïve CPS transformation: we can only capture the entire continuation, and can’t really inspect its structure. There is also no way to compose a captured continuation with the current continuation. And, in a naïve transformation, we would be constantly creating lots of heap allocation for these continuation closures; a push call effectively pushes a frame onto the heap as a closure, as we did above for `g'`.

There is also the question of when to perform the CPS transform; most optimizing compilers would like a large first-order graph to work on, which is out of step with the way CPS transformation breaks functions into many parts. Still, there is a nugget of wisdom here. What if we preserve the conventional compiler IR for most of the pipeline, and only perform the CPS transformation at the end? In that way we can have nice SSA-style optimizations. And, for return continuations of push calls, what if instead of allocating a closure, we save the continuation data on an explicit stack. As [Andrew Kennedy
notes](https://www.microsoft.com/en-us/research/wp-content/uploads/2007/10/compilingwithcontinuationscontinued.pdf), closures introduced by the CPS transform follow a stack discipline, so this seems promising; we would have:

```
(define (f'' k)
  (push! k)
  (push! h'')
  (g'' (lambda (x)
         (define h'' (pop!))
         (define k (pop!))
         (h'' k x))))

```

The explicit stack allows for generic slicing, which makes it a win for implementing delimited continuations.

### hoot and cps

Hoot takes the CPS transformation approach with stack-allocated return closures. In fact, Hoot goes a little farther, too far probably:

```
(define (f''')
  (push! k)
  (push! h''')
  (push! (lambda (x)
           (define h'' (pop!))
           (define k (pop!))
           (h'' k x)))
  (g'''))

```

Here instead of passing the continuation as an argument, we pass it on the stack of saved values. Returning pops off from that stack; for example, `(lambda () 42)` would transform as `(lambda () ((pop!) 42))`. But some day I should go back and fix it to pass the continuation as an argument, to avoid excess stack traffic for leaf function calls.

There are some gnarly details though, which I know you are here for!

#### splits

For our function `f`, we had to break it into two pieces: the part before the push-call to `g` and the part after. If we had two successive push-calls, we would instead split into three parts. In general, each push-call introduces a split; let us use the term *tails* for the components produced by a split. (You could also call them *continuations*.) How many tails will a function have? Well, one for the entry, one for each push call, and one any time control-flow merges between two tails. This is a fixpoint problem, given that the input IR is a graph. (There is also some special logic for `call-with-prompt` but that is too much detail for even this post.)

#### where to save the variables

Guile is a dynamically-typed language, having a uniform `SCM` representation for every value. However in the compiler and run-time we can often unbox some values, generally as `u64`/ `s64`/ `f64` values, but also raw pointers of some specific types, some GC-managed and some not. In native Guile, we can just splat all of these data members into 64-bit stack slots and rely on the compiler to emit stack maps to determine whether a given slot is a double or a tagged heap object reference or what. In WebAssembly though there is no sum type, and no place we can put either a `u64` or a `(ref eq)` value. So we have not one stack but three (!) stacks: one for numeric values, implemented using a Wasm memory; one for `(ref eq)` values, using a [table](https://webassembly.github.io/gc/core/syntax/modules.html#tables); and one for return continuations, because the `func` type hierarchy is disjoin from `eq`. It’s.... it’s gross? It’s gross.

#### what variables to save

Before a push-call, you save any local variables that will be live after the call. This is also a flow analysis problem. You can leave off constants, and instead reify them anew in the tail continuation.

I realized, though, that we have some pessimality related to stacked continuations. Consider:

```
(define (q x)
  (define y (f))
  (define z (f))
  (+ x y z))

```

Hoot’s CPS transform produces something like:

```
(define (q0 x)
  (save! x)
  (save! q1)
  (f))

(define (q1 y)
  (restore! x)
  (save! x)
  (save! y)
  (save! q2)
  (f))

(define (q2 z)
  (restore! x)
  (restore! y)
  ((pop!) (+ x y z)))

```

So `q0` saved `x`, fine, indeed we need it later. But `q1` didn’t need to restore `x` uselessly, only to save it again on `q2`‘s behalf. Really we should be applying a stack discipline for saved data within a function. Given that the source IR is a graph, this means another flow analysis problem, one that I haven’t thought about how to solve yet. I am not even sure if there is a solution in the literature, given that the SSA-like flow graphs plus tail calls / CPS is a somewhat niche combination.

#### calling conventions

The continuations introduced by CPS transformation have associated calling conventions: return continuations may have the generic varargs type, or the compiler may have concluded they have a fixed arity that doesn’t need checking. In any case, for a return, you call the return continuation with the returned values, and the return point then restores any live-in variables that were previously saved. But for a merge between tails, you can arrange to take the live-in variables directly as parameters; it is a direct call to a known continuation, rather than an indirect call to an unknown call site.

#### cps soup?

Guile’s intermediate representation is called [CPS
soup](https://wingolog.org/archives/2015/07/27/cps-soup), and you might wonder what relationship that CPS has to this CPS. The answer is not much. The continuations in CPS soup are first-order; a term in one function cannot continue to a continuation in another function. (Inlining and contification can merge graphs from different functions, but the principle is the same.)

It might help to explain that it is the same relationship as it would be if Guile represented programs using SSA: the Hoot CPS transform runs at the back-end of Guile’s compilation pipeline, where closures representations have already been made explicit. The IR is still direct-style, just that syntactically speaking, every call in a transformed program is a tail call. We had to introduce `save` and `restore` primitives to implement the saved variable stack, and some other tweaks, but generally speaking, the Hoot CPS transform ensures the run-time all-tail-calls property rather than altering the compile-time language; a transformed program is still CPS soup.

### fin

Did we actually make the right call in going for a CPS transformation?

I don’t have good performance numbers at the moment, but from what I can see, the overhead introduced by CPS transformation can impose some penalties, even 10x penalties in some cases. But some results are quite good, improving over native Guile, so I can’t be categorical.

But really the question is, is the performance acceptable *for the* *functionality*, and there I think the answer is more clear: we have a port of Fibers that I am sure Spritely colleagues will be writing more about soon, we have good integration with JavaScript promises while not relying on JSPI or Asyncify or anything else, and we haven’t had to compromise in significant ways regarding the source language. So, for now, I am satisfied, and looking forward to experimenting with the stack slicing proposal as it becomes available.

Until next time, happy hooting!

## related articles

- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [static single assignment for functional programmers](/archives/2011/07/12/static-single-assignment-for-functional-programmers)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)

Comments are closed.
