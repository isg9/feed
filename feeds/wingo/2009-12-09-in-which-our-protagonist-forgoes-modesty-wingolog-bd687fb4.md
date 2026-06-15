---
title: in which our protagonist forgoes modesty — wingolog
url: https://wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty
published: "2009-12-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty
---

# in which our protagonist forgoes modesty — wingolog

## [in which our protagonist forgoes modesty](/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty)

9 December 2009 8:51 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [meta-circular](/tags/meta-circular)
- [evaluator](/tags/evaluator)
- [compiler](/tags/compiler)

I wrote the most beautiful code of my life last week.

I would like to explain it to you all, but I have to tell a little bit of backstory first.

**the distant future, the year 2000**

Until earlier this year, [Guile](http://www.gnu.org/software/guile/) has been an interpreted Scheme. Guile ran your code by parsing it into tree-like data structures, then doing a depth-first evaluation of that tree. The evaluation itself was performed with a C function, `ceval`, which would recursively call itself when evaluating sub-expressions.

`ceval` was OK, but not so great. Instead of recursively traversing a tree, it's better to pre-examine the expressions you're going to evaluate, and then emit linear sequences of code to handle those steps. That's to say that it's better to have a compiler than an interpreter. So I dug up Keisuke Nishida's old bytecode compiler that he wrote back in 2001, and eventually hacked it into a shape that we merged it into Guile itself.

That was a pretty sweet hack, to retrofit a compiler into Guile. But it wasn't as beautiful as the code I wrote last week.

**the present**

See, the problem was that now we had two stacks: the C stack that `ceval` used, and the virtual machine stack used by byte-compiled code. This was a debugging headache, as to get backtraces you had to ping-pong back and forth between the two stacks, interleaving their frames together in the right order. Also with two stacks it's practically impossible to write a real debugger that does single-stepping, inspection and modification of stack frames, etc.

The two-stack solution (ahem) had another problem: you couldn't tail-call between interpreted and compiled code, because the procedures used different stacks. Normally this wasn't a big deal because all code was compiled, but it would occasionally bite you. (The usual case would be when you had a compiled Guile, but just pulled new code from the git repository, then tried to compile it again -- so some of your compiled code was out-of-date and therefore not loaded, you had a mix of compiled and interpreted code in some important places, and your loop starts consuming stack.)

Finally, interpreted code behaved differently than compiled code in some cases. For example, consider the following code:

```
;; Returns two values: the value, if found,
;; and a flag indicating success.
(define (table-lookup table key)
  (let ((handle (assq key table)))
    (if handle
        (values (cdr handle) #t)
        (values #f #f))))

(define (trace-call f . args)
  (let ((result (apply f args)))
    (format #t "\nfunction returned ~a\n" result)
    result))

(trace-call table-lookup '((x . y)) 'x)

```

So if I try this at my Guile 1.8 prompt, I get this:

```
guile> (trace-call table-lookup '((x . y)) 'x)
function returned #
$1 = y
$2 = #t

```

We see that `trace-call` returns two values, and the tracing printout shows a "multiple values object" -- a Scheme object like any other, but that the primitive `call-with-values` knows how to destructure. The toplevel repl is wrapped in a `call-with-values`, so `t` and `#t` print separately.

Now if I fire up Guile 1.9, let's see what we get:

```
> (trace-call table-lookup '((x . y)) 'x)
function returned y
$1 = y

```

Guile 1.9's repl compiles its expressions, by default, and indeed we see different behavior -- the trace printout has a naked value, `y`, and only one value is returned.

Both of these behaviors are compatible with standard Scheme from R5RS on. The origin of the difference is that the behavior of `values` within a continuation that was not created with `call-with-values` is unspecified. Relatedly, it is not specified what will happen when you return N values to a continuation accepting M values, where N != M.

What's happening is that the compiler actually has two return addresses in each stack frame -- one for the normal singly-valued case, and one for multiple values. `values` will return to the multiple-value return address (MVRA), and anything else will go to the normal return address. So actually, compiled code can choose what to do when it gets multiple values. Instead of raising an error when two values are returned to the `(let ((result ...)) ...)` continuation, Guile chooses to do what you (probably) expect and just drop the second value.

In contrast, with a C evaluator, even noticing that two values were returned to a singly-valued continuation is a pain -- because you have to check and branch every time you recursively call `ceval` to see if you're getting a multiple-values object.

But I digress. I promised something nice, and here I am noodling about something else.

**exit strategy**

The solution to all these problems, of course, is to use just one stack, and have that stack be the same as the one that compiled code uses.

Practically what this means is that `eval` should not be a C function, because Guile does not compile to C; it should be something that ends up as compiled code.

(For now, compiled code is bytecode, run on the VM. I'm being a little vague here because Guile doesn't do native compilation yet, but it will, within a year or two, and the same considerations apply.)

I actually toyed with the idea of writing a hand-coded `eval` in VM bytecode, but I came to my senses soon enough, and the answer was delightful.

**eval in scheme**

Of course! Scheme's `eval` should be written in Scheme itself. Then we just compile it to bytecode, like any other Scheme procedure.

At this point, anyone who's actually had to do Scheme at university (not me) will recognize this as the [meta-circular evaluator](http://en.wikipedia.org/wiki/Meta-circular_evaluator) pattern. To be honest I had never written one before -- and I think the reason was that they always seemed so peripheral. When you write a meta-circular evaluator, the language you really work in is the one that *implements* the meta-circular evaluator, not the one *implemented by* the evaluator -- or at least, that's the case if you're trying to get something done, rather than learn about language.

But this is different. This time the meta-circular evaluator actually sits at the heart of Guile -- in fact, we use `eval`, as implemented in Scheme, and compiled to bytecode, to compile the compiler -- which itself is written in Scheme of course.

In the end, though, you have to have a Scheme compiler to compile `eval.scm` itself, so we do end up keeping around an evaluator in C. Its only purpose is to interpret the compiler, so we can compile `eval.scm`: then the compiled version of `eval.scm` compiles the rest of Guile, including the compiler.

Another option would have been to require a new-enough version of Guile itself to compile the compiler. But I want to be able to [sanely bootstrap](http://www.doc.gold.ac.uk/~mas01cr/papers/s32008/sbcl.pdf) Guile's compiler, so that's out of the question. We could implement the compiler in portable Scheme, but that would forbid the compiler from making use of any of Guile's niceties.

**the code**

So [here it is](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/ice-9/eval.scm;h=e2746dce8cd0d5487f63ea87451b2631069b9e48;hb=HEAD#l103) (and below). I don't claim that it is actually the most elegant code I have written, though I can think of none better at the moment; nor is it the fastest code, nor the most concise. But it sits in such a powerful place, and in so few lines, that I cannot help but to be pleased with it.

```
(define primitive-eval
  (let ()
    ;; The "engine". EXP is a memoized expression.
    (define (eval exp env)
      (memoized-expression-case exp
        (('begin (first . rest))
         (let lp ((first first) (rest rest))
           (if (null? rest)
               (eval first env)
               (begin
                 (eval first env)
                 (lp (car rest) (cdr rest))))))

        (('if (test consequent . alternate))
         (if (eval test env)
             (eval consequent env)
             (eval alternate env)))

        (('let (inits . body))
         (let lp ((inits inits) (new-env (capture-env env)))
           (if (null? inits)
               (eval body new-env)
               (lp (cdr inits)
                   (cons (eval (car inits) env) new-env)))))

        (('lambda (nreq rest? . body))
         (let ((env (capture-env env)))
           (lambda args
             (let lp ((env env) (nreq nreq) (args args))
               (if (zero? nreq)
                   (eval body
                         (if rest?
                             (cons args env)
                             (if (not (null? args))
                                 (scm-error 'wrong-number-of-args
                                            "eval" "Wrong number of arguments"
                                            '() #f)
                                 env)))
                   (if (null? args)
                       (scm-error 'wrong-number-of-args
                                  "eval" "Wrong number of arguments"
                                  '() #f)
                       (lp (cons (car args) env)
                           (1- nreq)
                           (cdr args))))))))

        (('quote x)
         x)

        (('define (name . x))
         (define! name (eval x env)))

        (('apply (f args))
         (apply (eval f env) (eval args env)))

        (('call (f . args))
         (let ((proc (eval f env)))
           (let eval-args ((in args) (out '()))
             (if (null? in)
                 (apply proc (reverse out))
                 (eval-args (cdr in)
                            (cons (eval (car in) env) out))))))

        (('call/cc proc)
         (call/cc (eval proc env)))

        (('call-with-values (producer . consumer))
         (call-with-values (eval producer env)
           (eval consumer env)))

        (('lexical-ref n)
         (let lp ((n n) (env env))
           (if (zero? n)
               (car env)
               (lp (1- n) (cdr env)))))

        (('lexical-set! (n . x))
         (let ((val (eval x env)))
           (let lp ((n n) (env env))
             (if (zero? n)
                 (set-car! env val)
                 (lp (1- n) (cdr env))))))

        (('toplevel-ref var-or-sym)
         (variable-ref
          (if (variable? var-or-sym)
              var-or-sym
              (let lp ((env env))
                (if (pair? env)
                    (lp (cdr env))
                    (memoize-variable-access! exp (capture-env env)))))))

        (('toplevel-set! (var-or-sym . x))
         (variable-set!
          (if (variable? var-or-sym)
              var-or-sym
              (let lp ((env env))
                (if (pair? env)
                    (lp (cdr env))
                    (memoize-variable-access! exp (capture-env env)))))
          (eval x env)))

        (('module-ref var-or-spec)
         (variable-ref
          (if (variable? var-or-spec)
              var-or-spec
              (memoize-variable-access! exp #f))))

        (('module-set! (x . var-or-spec))
         (variable-set!
          (if (variable? var-or-spec)
              var-or-spec
              (memoize-variable-access! exp #f))
          (eval x env)))))

    ;; primitive-eval
    (lambda (exp)
      "Evaluate @var{exp} in the current module."
      (eval
       (memoize-expression ((or (module-transformer (current-module))
                                (lambda (x) x))
                            exp))
       '()))))

```

## related articles

- [maxwell's equations of software](/archives/2009/12/11/maxwells-equations-of-software)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [scheme quiz time!](/archives/2013/11/02/scheme-quiz-time)
- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 7 responses

1. [Rob Taylor](http://blog.floopily.org) says:[10 December 2009 9:25 AM](#b39b7bef76ebcc40f29d2c0bf2be0ac7fbdde562)

   binary solo!

2. bubo says:[10 December 2009 6:54 PM](#abfe59e953b9b71e25dc24ee3f9a53ca5882eb73)

   looks cool! is this actually \*really\* in the latest alpha tarball? how does it perform? nevermind! i have to go to the downlaod page inmeadeately...see ya later! ;)

3. [Elias](http://logforbuggymind.blogspot.com/) says:[10 December 2009 7:21 PM](#bf404a674c1c9814b65ff08dcd36c86fc78ce5b2)

   Bravo !! Hurray !

   loved it. I need to start poking into guile soon.

4. [Stephan Wehner](http://stephan.sugarmotor.org) says:[10 December 2009 10:32 PM](#bd765f51279943578e7c6e6156e5664605803c3b)

   You wrote "... so few lines ... " -- I've seen shorter programs :-)

   Seriously, you have any unit tests for that? It looks like a good example of write-once-code.

   Stephan

5. [Charles Stewart](http://www.advogato.org/person/chalst) says:[11 December 2009 8:52 AM](#13cd1b478504b66e7984506717504e4d0e98cb79)

   I posted a reply to advogato:

   http://www.advogato.org/person/chalst/diary/256.html

   BTW, I have the impression you occasionally read recentlog, but not so much that we can count on you catching a post there: is that right?

6. [wingo](http://wingolog.org/) says:[11 December 2009 2:25 PM](#c74a4854c8de7aa01ac39532f329fb4be60d2377)

   @Rob: Oh. Oh oh oh!

   As for the rest, thanks for the feedback. My reply ran overlong, so I made a separate post: [hither](http://wingolog.org/archives/2009/12/11/maxwells-equations-of-software). Thanks for stopping by :)

7. [Thomas Vander Stichele](http://thomas.apestaart.org) says:[12 December 2009 11:54 AM](#e87443bede60e8c82948a240c18a29cfa042fded)

   Even though I usually only get the musical references (are humans really dead ?) I still enjoy the read!

Comments are closed.
