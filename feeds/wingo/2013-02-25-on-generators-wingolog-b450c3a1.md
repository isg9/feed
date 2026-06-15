---
title: on generators — wingolog
url: https://wingolog.org/archives/2013/02/25/on-generators
published: "2013-02-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/02/25/on-generators
---

# on generators — wingolog

## [on generators](/archives/2013/02/25/on-generators)

25 February 2013 4:17 PM

- [igalia](/tags/igalia)
- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)
- [v8](/tags/v8)
- [javascriptcore](/tags/javascriptcore)
- [generators](/tags/generators)
- [delimited continuations](/tags/delimited%20continuations)
- [scheme](/tags/scheme)
- [guile](/tags/guile)

Hello! Is this dog?

![](//wingolog.org/pub/yes-this-is-dog.png)

Hi dog this is Andy, with some more hacking nargery. Today the topic is generators.

**(shift k)**

By way of introduction, you might recall (with a shudder?) a [couple](//wingolog.org/archives/2010/02/14/sidelong-glimpses) of [articles](//wingolog.org/archives/2010/02/26/guile-and-delimited-continuations) I wrote in the past on [delimited continuations](http://www.gnu.org/software/guile/manual/html_node/Prompts.html). I even gave a [talk on delimited continuations](//wingolog.org/pub/qc-2012-delimited-continuations-slides.pdf) at FrOSCon last year, a talk that was so hot that thieves broke into the the FrOSCon video squad's room and stole their hard drives before the edited product could hit the tubes. Not kidding! The FrOSCon folks still have the tapes somewhere, but meanwhile time marches on without the videos; truly, the terrorists have won.

Anyway, the summary of all that is that Scheme's much-praised `call-with-current-continuation` isn't actually a great building block for composable abstractions, and that delimited continuations have much better properties.

So imagine my surprise when Oleg Kiselyov, in a [mail to the Scheme standardization list](http://lists.scheme-reports.org/pipermail/scheme-reports/2012-February/001824.html), suggested a different replacement for `call/cc`:

> My bigger suggestion is to remove `call/cc` and `dynamic-wind` from the base library into an optional feature. I'd like to add the note inviting the discussion for more appropriate abstractions to supersede `call/cc` \-\- such \[as\] generators, for example.

He elaborated in [another mail](http://lists.scheme-reports.org/pipermail/scheme-reports/2012-February/001872.html):

> Generators are a wide term, some versions of generators are equivalent (equi-expressible) to shift/reset. Roshan James and Amr Sabry have written a [paper about all kinds of generators, arguing for the generator interface](http://www.cs.indiana.edu/~sabry/papers/yield.pdf).

And you know, generators are indeed a very convenient abstraction for composing producers and consumers of values -- more so than the lower-level delimited continuations. Guile's failure to standardize a generator interface is an instance of a class of Scheme-related problems, in which generality is treated as more important than utility. It's true that you can do generators in Guile, but we shouldn't make the user build their own.

The rest of the programming world moved on and built generators into their languages. James and Sabry's paper is good because it looks at `yield` as a kind of mainstream form of delimited continuations, and uses that perspective to place the generators of common languages in a taxonomy. Some of them are as expressive as delimited continuations. There are are also two common restrictions on generators that trade off expressive power for ease of implementation:

- Generators can be restricted to disallow backtracking, as in Python. This allows generators to yield without allocating memory for a new saved execution state, instead updating the stored continuation in-place. This is commonly called the *one-shot* restriction.

- Generators can also restrict access to their continuation. Instead of reifying generator state as external iterators, internal iterators call their consumers directly. The first generators developed by Mary Shaw in the Alphard language were like this, as are the generators in Ruby. This can be called the *linear-stack* restriction, after an implementation technique that they enable.

With that long introduction out of the way, I wanted to take a look at the proposal for generators in the latest ECMAScript draft reports -- to examine their expressiveness, and to see what implementations might look like. ES6 specifies generators with the the first kind of restriction -- with external iterators, like Python.

Also, Kiselyov & co. have a hip new paper out that explores the expressiveness of linear-stack generators, [*Lazy v. Yield: Incremental, Linear Pretty-printing*](http://okmij.org/ftp/continuations/PPYield/yield-pp.pdf). I was quite skeptical of this paper's claim that internal iterators were significantly more efficient than external iterators. So this article will also look at the performance implications of the various forms of generators, including full resumable generators.

**ES6 iterators**

As you might know, there's an official ECMA committee beavering about on a new revision of the ECMAScript language, and they would like to include both iterators and generators.

For the purposes of ECMAScript 6 (ES6), an iterator is an object with a `next` method. Like this one:

```
var p = console.log;

function integers_from(n) {
  function next() {
    return this.n++;
  }
  return { n: n, next: next };
}

var ints = integers_from(0);
p(ints.next()); // prints 0
p(ints.next()); // prints 1

```

Here, `int_gen` is an iterator that returns successive integers from 0, all the way up to infinity [2^53](http://www.jwz.org/blog/2010/10/every-day-i-learn-something-new-and-stupid/).

To indicate the end of a sequence, the current ES6 drafts follow Python's example by having the `next` method throw a special exception. ES6 will probably have an `isStopIteration()` predicate in the `@iter` module, but for now I'll just compare directly.

```
function StopIteration() {}
function isStopIteration(x) { return x === StopIteration; }

function take_n(iter, max) {
  function next() {
    if (this.n++ < max)
      return iter.next();
    else
      throw StopIteration;
  }
  return { n: 0, next: next };
}

function fold(f, iter, seed) {
  var elt;
  while (1) {
    try {
      elt = iter.next();
    } catch (e) {
      if (isStopIteration (e))
        break;
      else
        throw e;
    }
    seed = f(elt, seed);
  }
  return seed;
}

function sum(a, b) { return a + b; }

p(fold(sum, take_n(integers_from(0), 10), 0));
// prints 45

```

Now, I don't think that we can praise this code for being particularly elegant. The end result is pretty good: we take a limited amount of a stream of integers, and combine them with a fold, without creating intermediate data structures. However the means of achieving the result -- the implementation of the producers and consumers -- are nasty enough to prevent most people from using this idiom.

To remedy this, ES6 does two things. One is some "sugar" over the use of iterators, the `for-of` form. With this sugar, we can rewrite `fold` much more succinctly:

```
function fold(f, iter, seed) {
  for (var x of iter)
    seed = f(x, seed);
  return seed;
}

```

The other thing offered by ES6 are generators. Generators are functions that can `yield`. They are defined by `function*`, and yield their values using iterators:

```
function* integers_from(n) {
  while (1)
    yield n++;
}

function* take_n(iter, max) {
  var n = 0;
  while (n++ < max)
    yield iter.next();
}

```

These definitions are mostly equivalent to the previous iterator definitions, and are much nicer to read and write.

I say "mostly equivalent" because generators are closer to coroutines in their semantics because they can also receive values or exceptions from their caller via the `send` and `throw` methods. This is very much as in Python. [Task.js](http://taskjs.org/) is a nice example of how a coroutine-like treatment of generators can help avoid the [fraction-of-an-action](//wingolog.org/archives/2005/04/06/100) callback mess in frameworks like Twistednode.js.

There is also a `yield*` operator to delegate to another generator, as in [Python's PEP 380](http://www.python.org/dev/peps/pep-0380/).

OK, now that you know all the things, it's time to ask (and hopefully answer) a few questions.

**design of ES6 iterators**

Firstly, has the ES6 committee made the right decisions regarding the iterators proposal? For me, I think they did well in choosing external iterators over internal iterators, and the choice to duck-type the iterator objects is a good one. In a language like ES6, lacking delimited continuations or cross-method generators, external iterators are more expressive than internal iterators. The `for-of` sugar is pretty nice, too.

Of course, expressivity isn't the only thing. For iterators to be maximally useful they have to be fast, and here I think they could do better. Terminating iteration with a `StopIteration` exception is not so great. It will be hard for an optimizing compiler to rewrite the `throw`/ `catch` loop terminating condition into a simple `goto`, and with that you lose some visibility about your loop termination condition. Of course you can assume that the `throw` case is not taken, but you usually don't know for sure that there are no more `throw` statements that you might have missed.

[Dart did a better job here](https://www.dartlang.org/articles/m3-whats-new/iterables.html#iterator), I think. They split the `next` method into two: a `moveNext` method to advance the iterator, and a `current` getter for the current value. Like this:

```
function dart_integers_from(n) {
  function moveNext() {
    this.current++;
    return true;
  }
  return { current: n-1, moveNext: moveNext };
}

function dart_take_n(iter, max) {
  function moveNext() {
    if (this.n++ < max && iter.moveNext()) {
      this.current = iter.current;
      return true;
    }
    else
      return false;
  }
  return {
    n: 0,
    current: undefined,
    moveNext: moveNext
  };
}

function dart_fold(f, iter, seed) {
  while (iter.moveNext())
    seed = f(iter.current, seed);
}

```

I think it might be fair to say that it's easier to implement an ES6 iterator, but it is easier to use a Dart iterator. In this case, `dart_fold` is much nicer, and almost doesn't need any sugar at all.

Besides the aesthetics, Dart's `moveNext` is transparent to the compiler. A simple inlining operation makes the termination condition immediately apparent to the compiler, enabling range inference and other optimizations.

**iterator performance**

To measure the overhead of `for-of`-style iteration, we can run benchmarks on "desugared" versions of `for-of` loops and compare them to normal `for` loops. We'll throw in Dart-style iterators as well, and also a stateful closure iterator designed to simulate generators. (More on that later.)

I put up a [test case on jsPerf](http://jsperf.com/for-vs-iterators-vs-generators/2). If you click through, you can run the benchmarks directly in your browser, and see results from other users. It's imprecise, but I believe it is accurate, more or less.

The summary of the results is that current engines are able to run a basic summing `for` loop in about five cycles per iteration (again, assuming 2.5GHz processors; the Epiphany 3.6 numbers, for example, are mine). That's suboptimal only by a factor of 2 or so; pretty good. Thigh fives all around!

However, performance with iterators is not so good. I was surprised to find that most of the other tests are suboptimal by about a factor of 20. I expected the engines to do a better job at inlining.

One data point that stands out of my limited data set is the development Epiphany 3.7.90 browser, which uses JavaScriptCore from WebKit trunk. This browser did significantly better with Dart-style iterators, something I suspect due to better inlining, though I haven't verified it.

Better inlining and stack allocation is already a development focus of modern JS engines. Dart-style iterators will get faster without being a specific focus of optimization. It seems like a much better strategy for the ES6 specification to piggy-back off this work and specify iterators in such a way that they will be fast by default, without requiring more work on the exception system, something that is universally reviled among implementors.

**what about generators?**

It's tough to know how ES6 generators will perform, because they're not widely implemented and don't have a straightforward "desugaring". As far as I know, there is only one practical implementation strategy for `yield` in JavaScript, and that is to save the function's stack and the set of live registers. Resuming a generator copies the activation back onto the stack and restores the registers. Any other strategy that involves higher-level control-flow rewriting would make debugging too difficult. This is not a terribly onerous requirement for an implementation. They already have to know exactly what is live and what is not, and can reconstruct a stack frame without too much difficulty.

In practice, invoking a generator is not much different from calling a stateful closure. For this reason, I added another test to the [jsPerf benchmark](http://jsperf.com/for-vs-iterators-vs-generators/2) I mentioned before that tests stateful closures. It does slightly worse than iterators that keep state in an object, but not much worse -- and there is no essential reason why it should be slow.

On the other hand, I don't think we can expect compilers to do a good job at inlining generators, especially if they are specified as terminating with a `StopIteration`. Terminating with an exception has its own quirks, as [Chris Leary wrote a few years ago](http://blog.cdleary.com/2010/12/a-generator-protocol-trap/), though I think in his conclusion he is searching for virtues where there are none.

**dogs are curious about multi-shot generators**

So dog, that's ECMAScript. But since we started with Scheme, I'll close this parenthesis with a note on resumable, multi-shot generators. What is the performance impact of allowing generators to be fully resumable?

Being able to snapshot your program at any time is pretty cool, but it has an allocation cost. That's pretty much the dominating factor. All of the examples that we have seen so far execute (or should execute) without allocating any memory. But if your generators are resumable -- or even if they aren't, but you build them on top of full delimited continuations -- you can't re-use the storage of an old captured continuation when reifying a new one. A test of iterating through resumable generators is a test of short-lived allocations.

I ran this test in my Guile:

```
(define (stateful-coroutine proc)
  (let ((tag (make-prompt-tag)))
    (define (thunk)
      (proc (lambda (val) (abort-to-prompt tag val))))
    (lambda ()
      (call-with-prompt tag
                        thunk
                        (lambda (cont ret)
                          (set! thunk cont)
                          ret)))))

(define (integers-from n)
  (stateful-coroutine
   (lambda (yield)
     (let lp ((i n))
       (yield i)
       (lp (1+ i))))))

(define (sum-values iter n)
  (let lp ((i 0) (sum 0))
    (if (< i n)
        (lp (1+ i) (+ sum (iter)))
        sum)))

(sum-values (integers-from 0) 1000000)

```

I get the equivalent of 2 Ops/sec (in terms of the jsPerf results), allocating about 0.5 GB/s (288 bytes/iteration). This is similar to the allocation rate I get for JavaScriptCore, which is also not generational. V8's generational collector lets it sustain about 3.5 GB/s, at least in my tests. [Test case here](http://jsperf.com/allocation-rate); multiply the 8-element test by 100 to yield approximate MB/s.

Anyway these numbers are much lower than the iterator tests. ES6 did right to specify one-shot generators as a primitive. In Guile we should perhaps consider implementing some form of one-shot generators that don't have an allocation overhead.

Finally, while I enjoyed the paper from Kiselyov and the gang, I don't see the point in praising simple generators when they don't offer a significant performance advantage. I think we will see external iterators in JavaScript achieve near-optimal performance within a couple years without paying the expressivity penalty of internal iterators.

Catch you next time, dog!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [generators in v8](/archives/2013/05/08/generators-in-v8)
- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [eval, that spectral hound](/archives/2012/02/01/eval-that-spectral-hound)
- [value representation in javascript implementations](/archives/2011/05/18/value-representation-in-javascript-implementations)

### 4 responses

1. [Vyacheslav Egorov](http://mrale.ph) says:[25 February 2013 10:27 PM](#b0e6f466e7fc0dc6d7a98405b5b1126b5e4264fe)

   I'd replace "stack allocation" with "scalar replacement". Having an iterator allocated on the stack in the otherwise heterogeneous heap would be quite a pain (and more pain when moving it out to the heap when surrounding code deopts, oh ah). Scalar replacement (along the lines of LuaJIT2 "allocation sinking") on the other hand seems exactly what the doctor prescribed here

2. Mitchell N Charity says:[28 February 2013 5:33 AM](#ae212ff662e11ae1eea65d357a52b079d458a35b)

   > I expected the engines to do a better job at inlining.

   Perhaps, for v8 on ia64 and arm, it's the heap-allocated variables?

   As you observed [in 2011](http://wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler):

   > Some conditions that must currently be met for inlining to proceed are: \[...\] 2. Neither the inner nor the outer functions may contain heap-allocated variables. (In practice, this means that neither function may contain lexical closures.)

   Here in the future, unless you're on ia32, it's still deadly: [hydrogen.cc](https://github.com/v8/v8/blob/686b0a458ebd5189fd595e0d1eb15f257abb54f7/src/hydrogen.cc#L7516) . Sigh.

3. [qznc](http://beza1e1.tuxen.de) says:[28 February 2013 12:07 PM](#90f7c305161de9a0f3dc42eed4afe4b6b8988377)

   The comparable feature in D would be ranges (http://dlang.org/phobos/std\_range.html). They are kind of a generalization of iterators and defined as a family of interfaces.

   Multi-shot generators would be a "ForwardRange" for example. It provides a save() function for backtracking.

4. [eddyb](https://github.com/eddyb) says:[11 April 2013 5:07 PM](#4c922aae3e7f435830486b3e12f68dcdad3e01c4)

   traceur-compiler has been able to desugar generators into a state-machine for months now. Also, if you want ES6, you should try it ;).

Comments are closed.
