---
title: a lambda is not (necessarily) a closure — wingolog
url: https://wingolog.org/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure
published: "2016-02-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure
---

# a lambda is not (necessarily) a closure — wingolog

## [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

8 February 2016 10:12 AM

- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [scheme](/tags/scheme)
- [closure optimization](/tags/closure%20optimization)
- [contification](/tags/contification)

**preface**

Greets, folks! Check it out: Guile had a whole track devoted to it at FOSDEM this year. OK, so it was only half a day, but [there were like a dozen talks!](https://fosdem.org/2016/schedule/track/gnu_guile/) And the room was full all the morning! And -- get this -- I had *nothing* to do with its organization! I think we can credit the [Guix](https://gnu.org/s/guix/) project with the recent surge of interest in Guile; fully half the talks were from people excited about using Guix to solve their problems. Thanks very, very much to [Pjotr Prins](http://thebird.nl/) for organizing the lovely event.

I gave a [talk on how the Guile 2.2 compiler and virtual machine could change the way people program](https://fosdem.org/2016/schedule/event/guilevm/). Happily, the video recording came out OK! Video below ( [or here if that doesn't work](https://wingolog.org/pub/fosdem-2016-guile.mp4)), and slides [here](https://wingolog.org/pub/fosdem-2016-guile-slides.pdf).

[Click to download video](https://wingolog.org/pub/fosdem-2016-guile.mp4)

The time was super-limited though and I wasn't able to go into the detail that I'd like. So, dear readers, here we are, with a deeper look on `lambda` representation in Guile.

**a lambda is not (necessarily) a closure**

What is this?

```
(lambda (a b) (+ a b))

```

If you answer, "it's a lambda expression", you're right! You're also right if you say it's a function -- I mean, `lambda` makes a function, right? There are lots of things that you could say that would be right, including silly things like "twenty-two characters set in an awkward typeface".

But if you said "it's a closure" -- well you're right in general I guess, like on a semantic what-does-it-mean level, but as far as how Guile represents this thing at run-time, hoo boy are there a number of possibilities, and a closure is just one of them. This article dives into the possibilities, with the goal being to help you update your mental model of "how much do things cost".

In Guile, a `lambda` expression can be one of the following things at run-time:

1. Gone

2. Inlined

3. Contified

4. Code pointer

5. Closure

Let's look into these one-by-one.

**lambda: gone**

If Guile can prove that a `lambda` expression is never reached, it won't be present at run-time. The main way this happens is via [partial evaluation](https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile), but later passes can do this too. In the most basic example, consider the `lambda` bound to `f` by this `let` expression.

```
(let ((f (lambda ()
           (launch-the-missiles!))))
  42)

```

Guile has an `,optimize` command that can be run at the REPL to show the effect of partial evaluation on your code. These days it's a bit out of date in a way -- it can't show what CPS-based optimization will do to your code -- but for our purposes here it will transform the expression to the following code:

```
(let ((f (lambda ()
           (launch-the-missiles!))))
  42)
=> 42

```

So the `lambda` is gone, big whoop. The interesting thing though is that this happens concurrently with other things that partial evaluation does, so the `lambda` goes away in this expression too:

```
(let ((launch? #f)
      (f (lambda ()
           (launch-the-missiles!))))
  (if launch? (f) 'just-kidding))
=> 'just-kidding

```

**lambda: inlined**

The other trick that partial evaluation can do with `lambda` expressions is inlining. Re-taking the example above, if we change `launch?` to `#t`, the branch folds the other way and the application `(f)` inlines:

```
(let ((launch? #t)
      (f (lambda ()
           (launch-the-missiles!))))
  (if launch? (f) 'just-kidding))
=> (let ((launch? #t)
         (f (lambda ()
              (launch-the-missiles!))))
     (if #t (f) 'just-kidding))
=> (let ((launch? #t)
         (f (lambda ()
              (launch-the-missiles!))))
     (f))
=> (let ((launch? #t)
         (f (lambda ()
              (launch-the-missiles!))))
     ((lambda () (launch-the-missiles!))))
=> (let ((launch? #t)
         (f (lambda ()
              (launch-the-missiles!))))
     (launch-the-missiles!))
=> (launch-the-missiles!)

```

Here again the `lambda` is gone, but not because it was unreachable, but because it was inlined into its use. I showed some intermediate steps as well, just so you get a feel about how partial evaluation works. The inlining step is illustrated by the fourth transformation, where the `lambda` application went away, replaced by its body.

Partial evaluation can also unroll many kinds of recursion:

```
(letrec ((lp (lambda (n)
               (if (zero? n)
                   n
                   (+ n (lp (1- n)))))))
  (lp 5))
=> 15

```

The partial evaluator in Guile 2.2 is more or less unchanged from the one in Guile 2.0, so you get these benefits on old Guile as well. Building a good intuition as to what the partial evaluator will do is important if you want to get the best performance out of Guile. Use the `,optimize` command at the REPL to see the effects of partial evaluation on any given expression.

**lambda: contified**

So, here we step into the unknown, in the sense that from here on out, these optimizations are new in Guile 2.2. Unfortunately, they can be hard to see as they aren't really representable in terms of source-to-source transformations over Scheme programs. Consider this program:

```
(define (count-down n)
  (define loop
    (lambda (n out)
      (let ((out (cons n out)))
        (if (zero? n)
            out
            (loop (1- n) out)))))
  (loop n '()))

```

It's a little loop that builds a list of integers. The `lambda` in this loop, bound to `loop`, will be *contified* into the body of `count-down`.

To see that this is the case, we have to use a new tool, `,disassemble` (abbreviated `,x`). This takes a procedure and prints its bytecode. It can be hard to understand, so I'm going to just point out some "shapes" of disassembly that you can recognize.

```
> ,x count-down
Disassembly of # at #x9775a8:

[...]
L1:
  10    (cons 2 1 2)
  11    (br-if-u64-=-scm 0 1 #f 5) ;; -> L2
  14    (sub/immediate 1 1 1)
  15    (br -5)                    ;; -> L1
L2:
[...]

```

I've snipped the disassembly to the interesting part. The first thing to notice is that there's just one procedure here: only one time that `,x` prints "Disassembly of ...". That means that the lambda was eliminated somehow, either because it was dead or inlined, as described above, or because it was contified. It wasn't dead; we can see that from looking at the `,optimize` output, which doesn't significantly change the term. It wasn't inlined either; again, `,optimize` can show you this, but consider that because partial evaluation can't determine when the loop would terminate, it won't find a point at which it can stop unrolling the loop. (In practice what happens though is that it tries, hits an effort or code growth limit, then aborts the inlining attempt.)

However, what we see in the disassembly is the body of the loop: we cons something onto a list (the `cons`), check if a two numbers are equal ( `br-if-u64-=-scm`), and if they are we jump out of the loop ( `L2`). Otherwise we subtract 1 from a number ( `sub/immediate`) and loop ( `br` to `L1`). That is the loop. So what happened?

Well, if inlining is copying, then contification is rewiring. Guile's compiler was able to see that although it couldn't inline the `loop` function, it could see all of `loop`'s callers, and that `loop` always returned to the same "place". (Another way to say this is that `loop` is always called with the same continuation.) The compiler was then able to incorporate the body of `loop` into `count-down`, rewiring calls to `loop` to continue to `loop`'s beginning, and rewriting returns from `loop` to proceed to the continuation of the `loop` call.

**a digression on language**

These words like "contification" and "continuation" might be unfamiliar to you, and I sympathize. If you know of a better explanation of contification, I welcome any links you might have. The name itself comes from a particular formulation of the intermediate language used in Guile, the so-called "CPS" language. In this language, you convert a program to make it so it never returns: instead, each sub-expression passes its values to its *continuation* via a tail call. Each continuation is expressed as a `lambda` expression. See [this article](https://wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers) for an intro to CPS and how it relates to things you might know like SSA.

Transforming a program into CPS explodes it into a bunch of little lambdas: every subexpression gets its own. You would think this would be a step backwards, if your goal is to eliminate closures in some way. However it's possible to syntactically distinguish between `lambda` expressions which are only ever used as continuations and those that are used as values. Let's call the former kind of `lambda` a *cont* and the latter a *function*. A cont- `lambda` can be represented at run-time as a label -- indeed, the disassembly above shows this. It turns out that all `lambda` expressions introduced by the CPS transformation are conts. Conts form a first-order flow graph, and are basically the same as SSA basic blocks. If you're interested in this kind of thing, see Andrew Kennedy's great paper, [Compiling with Continuations, Continued](http://research.microsoft.com/pubs/64044/compilingwithcontinuationscontinued.pdf), and see also [CPS soup](https://wingolog.org/archives/2015/07/27/cps-soup) for more on how this evolved in Guile 2.2.

I say all this to give you a vocabulary. Functions that are present in the source program start life as being treated as function- `lambda` s. Contification takes function- `lambda` values and turns then into cont- `lambda` labels, if it can. That's where the name "contification" comes from. For more on contification, see [MLton's page on its contification pass](http://mlton.org/Contify), linking to the original paper that introduces the concept.

**and we're back**

Contification incorporates the body of a function into the flow graph of its caller. Unlike inlining, contification is *always* an optimization: it never causes code growth, and it enables other optimizations by exposing first-order control flow. (It's easier for the compiler to reason about first-order loops than it is to reason about control flow between higher-order functions.)

Contification is a reliable optimization. If a function's callers are always visible to the compiler, and the function is always called with the same continuation, it will be contified. These are two fairly simple conditions that you can cultivate your instincts to detect and construct.

Contification can also apply to mutually recursive functions, if as a group they are all always called with the same continuation. It's also an iterative process, in the sense that contifying one set of functions can expose enough first-order control flow that more contification opportunities become apparent.

It can take a while to get a feel for when this optimization applies. You have to have a feel for what a continuation is, and what it means for a function's callers to all be visible to the compiler. However, once you do internalize these conditions, contification is something you can expect Guile's compiler to do to your code.

**lambda: code pointer**

The next representation a `lambda` might have at run-time is as a code pointer. In this case, the function fails the conditions for contification, but we still avoid allocating a closure.

Here's a little example to illustrate the case.

```
(define (thing)
  (define (log what)
    (format #t "Very important log message: ~a\n" what))
  (log "ohai")
  (log "kittens")
  (log "donkeys"))

```

In this example, `log` is called with three different continuations, so it's not eligible for contification. Unfortunately, this example won't illustrate anything for us because the `log` function is so small that partial evaluation will succeed in inlining it. (You could determine this for yourself by using `,optimize`.) So let's make it bigger, to fool the inliner:

```
(define (thing)
  (define (log what)
    (format #t "Very important log message: ~a\n" what)
    ;; If `log' is too short, it will be inlined.  Make it bigger.
    (format #t "Did I ever tell you about my chickens\n")
    (format #t "I was going to name one Donkey\n")
    (format #t "I always wanted a donkey\n")
    (format #t "In the end we called her Raveonette\n")
    (format #t "Donkey is not a great name for a chicken\n")
    (newline) (newline) (newline) (newline) (newline))
  (log "ohai")
  (log "kittens")
  (log "donkeys"))

```

Now if we disassembly it, we do get disassembly for two different functions:

```
,x thing
Disassembly of # at #x97d704:
[...]

Disassembly of log at #x97d754:
[...]

```

So, good. We defeated the inliner. Let's look closer at the disassembly of the outer function.

```
,x thing
Disassembly of # at #x97d704:
[...]
  12    (call-label 3 2 8)              ;; log at #x97d754

```

Here we see that instead of the generic `call` instruction, we have the specific `call-label` instruction which calls a procedure whose code is at a known offset from the calling function.

`call-label` is indeed a cheaper call than the full `call` instruction that has to check that the callee is actually a function and so on. But that's not the real optimization here. If all callers of a function are known -- and by this time, you're starting to catch the pattern, I think -- if all callers are known, then *the procedure does not need to exist as a value at run-time*.

This affords a number of optimization opportunities. Theoretically there are many -- all call sites can be specialized to the specific callee. The callee can have an optimized calling convention that doesn't have anything to do with the generic convention. Effect analysis can understand the side effects and dependencies of the callee in a more precise way. The compiler can consider unboxing some arguments and return values, if it finds that useful.

In Guile though, there's only one real optimization that we do, and that is related to free variables. Currently in Guile, all procedures have a uniform calling convention, in which the procedure being called (the callee) is itself passed as the zeroeth argument, and then the arguments follow on the stack. The function being called accesses its free variables through that zeroeth argument. If however there is no need for the procedure to be represented as a value, we are free to specialize that zeroeth argument.

So, consider a *well-known* procedure like `log` above. (By "well-known", we mean that all of `log`'s callers are known.) Since `log` doesn't actually have any lexically bound free variables, we can just pass in anything as argument zero when invoking it. In practice we pass `#f`, because it happens to be an easy value to make.

(Why isn't `format` treated as a free variable in `log`? Because there is special support from the linker for lazily initializing the locations of variables imported from other modules or defined at the top level instead of within a lexical contour. In short: only variables that are (a) used within the `lambda` and (b) defined within a `let` or similar count towards a `lambda`'s free variables.)

For a well-known procedure with only one free variable, we can pass in that free variable as the zeroeth argument. Internally to the function, we rewrite references to that free variable to reference argument 0 instead. This is a neat hack because we can have a `lambda` with a free variable but which results in no allocation at run-time.

Likewise if there are two free variables -- and this is starting to sound like Alice's restaurant, isn't it -- well we do have to pass in their values to the procedure, but we don't have to build an actual closure object with a tag and a code pointer and all. Pairs happen to be small and have some fast paths in Guile, so we use that. References to the free variables get internally rewritten to be `car` or `cdr` of argument 0.

For three or more free variables, we do the same, but with a vector.

For a final trick, a set of mutually recursive procedures whose callers are all known can share the object that collects their free variables. We collect the union of the free variables of all of the procedures, and pack them into a specialized representation as above.

Note that for well-known procedures, all variables that are free in the `lambda` are also free in the caller; that's why the 1-free-variable substitution works. The `lambda` is bound in a scope that dominates its callers, but its free variables dominate the `lambda` so they dominate the callers too. For that reason in this case we could choose to do lambda lifting instead, with no penalty: instead of bundling up the free variables in a heap object, we could pass them as arguments. Dybvig claims this is not a great idea because it increases register pressure. That could be true, but I haven't seen the numbers. Anyway, we do the flat closure thing, so we pack the free vars into data.

All these ideas came pretty much straight from the great [Optimizing Closures in O(0) Time](http://users-cs.au.dk/danvy/sfp12/papers/keep-hearn-dybvig-paper-sfp12.pdf) by Andrew Keep et al.

**lambda: closure**

OK! So you have a `lambda` whose callees are not all visible to the compiler. You need to reify the procedure as a value. That reified procedure-as-value is a *closure*: an object with a tag, a code pointer, and an array of free variables.

Of course, if the procedure has no free variables, you just have the tag and the code pointer... and because Scheme is semantically squirrely when it comes to the result of `(eqv? (lambda () 10) (lambda () 10))` (it's unspecified: `lambda` expressions don't have identity), we can statically allocate the closure in the binary, as a constant.

Otherwise we do allocate the heap object.

Note however that if a group of mutually recursive procedures has just one entry that is not "well-known", then that procedure clique can share one closure object.

**lambda: it's complicated**

In summary, a lambda is an abstraction that has many concrete representations. Guile will choose the cheapest representation that it can. If you need to eke out even more performance from your program, having a good mental model of how the abstract maps to the concrete will help you know where to focus your efforts, and what changes might be helpful. Good luck, and happy hacking!

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 4 responses

1. Kevin Millikin says:[8 February 2016 12:00 PM](#a6aa65d752212264490c4b36f69f0ab8bbb4a3c0)

   Thanks for this post, Andy. It drives me crazy when people talk about "closures" as a language feature. It's an implementation technique. "Pass a closure" is a category error. (See my first response at http://lambda-the-ultimate.org/node/2009).

   Just say "pass a function".

   (I wouldn't say 'lambda' either, though.)

2. [John Cowan](http://www.ccil.org/~cowan) says:[8 February 2016 10:24 PM](#44b28cd8f51dcc9aa6a938912fba769ced7cd628)

   You know, if one person, just one person, does it, they may think he's really dangerous and they won't flame him.

   And if two people do it, in harmony, they may think they're both Perl hackers and they won't flame either of them.

   And if three people do it! Can you imagine three people loggin' in, singin' a bar of \`\`Alice's Usenet Flame'' and loggin' out? They may think it's a re-implementation of sendmail!

   —me, many years ago

3. says:[9 February 2016 6:14 AM](#37a60ec488389fd50e0037ad0572e4d88942139e)

   Kevin: Okay, but "first-class functions with lexically-scoped free variables" is a bit of a mouthful.

4. Alex Vong says:[23 February 2016 3:57 AM](#4859cc6f81b4b40b327e5bbf030e68eaef989151)

   Thanks for your talk! I have learned scheme (using guile) for one year now, I would love to learn how compiler works in general. IMO, scheme is the perfect language since it has no awkward syntax to worry about, everything is a s-exp. I am going to watch the video now! One thing though, could you provide the video in webm format as well? As we know mpeg-4 is a patented format, I really preferred the patent-free webm format (https://www.fsf.org/news/supporting-webm). Thanks again.

Comments are closed.
