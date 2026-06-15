---
title: scheme quiz time! — wingolog
url: https://wingolog.org/archives/2013/11/02/scheme-quiz-time
published: "2013-11-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/11/02/scheme-quiz-time
---

# scheme quiz time! — wingolog

## [scheme quiz time!](/archives/2013/11/02/scheme-quiz-time)

2 November 2013 1:42 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [eval](/tags/eval)
- [meta-circular](/tags/meta-circular)
- [continuations](/tags/continuations)

Scheme quiz time!

Consider the following two functions:

```
(define (test1 get)
  (let ((v (make-vector 2 #f)))
    (vector-set! v 0 (get 0))
    (vector-set! v 1 (get 1))
    v))

(define (test2 get)
  (let* ((a (get 0))
         (b (get 1)))
    (vector a b)))

```

Assume the usual definitions for all of the free variables like `make-vector` and so on. These functions both create a vector with two elements. The first element is the result of a call to `(get 0)`, where `get` is a function the user passes in as an argument. Likewise the second comes from `(get 1)`.

```
(test1 (lambda (n) n)) => #(0 1)
(test2 (lambda (n) n)) => #(0 1)

```

So the functions are the same.

Or are they?

Your challenge: write a standard Scheme function `discriminate` that, when passed either `test1` or `test2` as an argument, can figure out which one it is.

. . .

Ready? If you know Scheme, you should think on this a little bit before looking at the answer. I'll wait.

. . .

OK!

We know that in both functions, two calls are made to the `get` function, in the same order, and so really there should be no difference whatsoever.

However there is a difference in the *continuations* of the `get` calls. In `test1`, the continuation includes the identity of the result vector -- because the vector was allocated before the `get` calls. On the other hand `test2` only allocates the result after the calls to `get`. So the trick is just to muck around with continuations so that you return twice from a call to the test function, and see if both returns are the same or not.

```
(define (discriminate f)
  (let ((get-zero-cont #t)
        (first-result #f))
    (define (get n)
      (when (zero? n)
        (call/cc (lambda (k)
                   (set! get-zero-cont k))))
      n)
    (let ((result (f get)))
      (cond
       (first-result
        (eq? result first-result))
       (else
        (set! first-result result)
        (get-zero-cont))))))

```

In the call to `f`, we capture the continuation of the entry to the `(get 0)` call. Then later we re-instate that continuation, making the call to `f` return for a second time. Then we see if both return values are the same object.

```
(discriminate test1) => #t
(discriminate test2) => #f

```

If they are the same object, then the continuation captured the identity of the result vector -- and if not, the result was only allocated after the `get` calls.

**so what?**

Unhappily, this has practical ramifications. In many compilers it would be advantagous to replace calls to `vector` with calls to `make-vector` plus a series of `vector-set!` operations. Such a transformation lowers the live variable pressure. If you have a macro that generates a bison-like parser whose parse table is built by a call to `vector` with 400 arguments -- this happens -- you'd rather not have 400 live variables in the function that builds that table. But this isn't a safe transformation to make, unless you can prove that no argument captures the current continuation. Happily, for the parser generator macro this is the case, but it's not something to bet on.

It gets worse, though. Since `test1` returns the same object, it is possible to use continuations to mutate previous return values, with nary a `vector-set!` in sight!

```
(define (discriminate2 f)
  (let ((get-zero-cont #f)
        (escape #f))
    (define (get n)
      (case n
        ((0) (call/cc (lambda (k)
                        (set! get-zero-cont k)
                        0)))
        ((1) (if escape
                 (escape)
                 1))))
    (let ((result (f get)))
      (call/cc
       (lambda (k)
         (set! escape k)
         (get-zero-cont 42)))
      result)))

(discriminate2 test1) => #(42 1)
(discriminate2 test2) => #(0 1)

```

This... this is crazy.

**story time**

Now it's story time. Guile has a bytecode VM, and usually all code is compiled to that VM. But it also has an interpreter, for various purposes, and that interpreter is fairly classic: [it's a recursive function that takes a "memoized expression" and an environment as parameters](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty). Only, the environment was silly -- it was just a list of values. Before evaluating, a "memoizer" runs to resolve lexical references to indexes in that list, and entering a new lexical contour conses on that list.

Well of course that makes lexical variable lookup expensive. It usually doesn't matter as everything is compiled, but it's a bit shameful, so I rewrote it recently to use two-dimensional environments. Let me drop some ASCII on you:

```
   +------+------+------+------+------+
   | prev |slot 0|slot 1| ...  |slot N|
   +------+------+------+------+------+
      \/
   +------+------+------+------+------+
   | prev |slot 0|slot 1| ...  |slot N|
   +------+------+------+------+------+
      \/
     ...
      \/
   toplevel

```

It's a chain of vectors, linked through their first elements. Resolving a lexical in this environment has two dimensions, the depth and the width.

Vectors.

You see where I'm going with this?

I implemented the "push-new-environment" operation as a sequence of `make-vector` and an `eval`-and- `vector-set!` loop. Here's the actual clause that implements this:

```
(define (eval exp env)
  (match exp
    ...
    (('let (inits . body))
     (let* ((width (vector-length inits))
            (new-env (make-env width #f env)))
       (let lp ((i 0))
         (when (< i width)
           (env-set! new-env 0 i (eval (vector-ref inits i) env))
           (lp (1+ i))))
       (eval body new-env)))
    ...))

```

This worked fine. It was fast, and correct. Or so I thought. I used this interpreter to bootstrap a fresh Guile compile and all was good. Until running those damned test suites that use `call/cc` to return multiple times from `let` initializers, as in my `discriminate2` test. While the identity of `env` isn't visible to a program as such, the ability of `call/cc` to peel apart allocation and initialization of the environment vector makes this particular implementation strategy not viable.

In the end I'll inline a few arities, and then have a general case that allocates heap storage for the temporaries:

```
(case (vector-length env)
  ((0) (vector env))
  ((1) (vector env (eval (vector-ref inits 0) env)))
  ...
  (else
   (list->vector
    (cons env
          (map (lambda (x) (eval x env))
               (vector->list inits))))))

```

Of course I'd use a macro to generate that. It's terrible, but oh well. *Es lo que hay*.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [guile and delimited continuations](/archives/2010/02/26/guile-and-delimited-continuations)
- [sidelong glimpses](/archives/2010/02/14/sidelong-glimpses)
- [maxwell's equations of software](/archives/2009/12/11/maxwells-equations-of-software)
- [in which our protagonist forgoes modesty](/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

### 5 responses

1. [amoe](http://www.solasistim.net/) says:[2 November 2013 1:51 PM](#77b056b09d596d971e4f71bda22632e0d9d6348d)

   Great quiz. I failed. Didn't even have any idea, let alone being able to write that. Wizardry.

2. Jason Orendorff says:[3 November 2013 3:59 AM](#cf07c4223b3d071eb094bf5f4fd17ccb6a55976d)

   Heh! I got the quiz right away, but reading the story I think I would have totally walked right into that one too...

3. [Jeff Walden](http://whereswalden.com/) says:[3 November 2013 4:24 AM](#e2d82777068f71834d3fb92d392bec0a333add6d)

   Took me a few minutes' staring, then a brief nap, then another few minutes' staring, plus some r5rs consultation, but I got it in the end. :-) It's been a long time since I wrote any Scheme (interestingly, I think the last time I did so I actually *was* using call/cc), so I'm actually a little surprised I twigged to it.

   call/cc is powerful enough that, interesting as its uses may be, I'd argue against it in any language I was implementing with input into its design. It's worth knowing about the idea and working through its implications to enhance mental dexterity. But it's Pandora's box in the presence of mutation, and even in functional languages it's not always right to be forced to avoid that.

4. [Nala Ginrut](http://nalaginrut.com) says:[28 July 2015 5:08 PM](#e412db97570eb649bdc2000a5ecdae04bb9e3a8b)

   I read it several times. Now my understand is that:

   1\. In test1, since vector exists before continuation capturing, so it won't be created twice when continuation was reinstated in the second calling of \`get'.

   2\. In test2, the vector was created again when reinstating in the second \`get', so the first returned vector is not identity with the second one (compared with eq?).

   Well, I think this would only happen when we capture the continuation, right? Or it's so inefficient...

5. [Nala Ginrut](http://nalaginrut.com) says:[28 July 2015 5:12 PM](#0e87c16dfd48f40a7a8c58fd6895f2a26e4a9f27)

   And if you use \`equal?' instead of \`eq?' when comparing first-result and result, both results are all #t.

Comments are closed.
