---
title: eval, that spectral hound — wingolog
url: https://wingolog.org/archives/2012/02/01/eval-that-spectral-hound
published: "2012-02-01T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/02/01/eval-that-spectral-hound
---

# eval, that spectral hound — wingolog

## [eval, that spectral hound](/archives/2012/02/01/eval-that-spectral-hound)

1 February 2012 3:33 PM

- [scheme](/tags/scheme)
- [local-eval](/tags/local-eval)
- [fexprs](/tags/fexprs)
- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)
- [igalia](/tags/igalia)
- [guile](/tags/guile)

Friends, I am not a free man. Eval has been my companion of late, a hellhound on my hack-trail. I give you two instances.

**the howl of the-environment, across the ages**

As legend has it, in the olden days, Aubrey Jaffer, the duke of [SCM](http://people.csail.mit.edu/jaffer/SCM), introduced low-level [FEXPR](http://en.wikipedia.org/wiki/Fexpr)-like macros into his Scheme implementation. These allowed users to capture the lexical environment:

```
(define the-environment
  (procedure->syntax
   (lambda (exp env)
     env)))

```

Tom Lord inherited this cursed bequest from Jaffer, when he established himself in the nearby earldom of Guile. It so affected him that he added [`local-eval`](http://www.gnu.org/software/guile/docs/docs-1.8/guile-ref/Local-Evaluation.html#index-local_002deval-2270) to Guile, allowing the user to evaluate an expression within a captured local environment:

```
(define env (let ((x 10)) (the-environment)))
(local-eval 'x env)
=> 10
(local-eval '(set! x 42) env)
(local-eval 'x env)
=> 42

```

Since then, the tenants of the earldom of Guile have been haunted by this strange leakage of the state of the interpreter into the semantics of Guile.

When the Guile co-maintainer title devolved upon me, I had a plan to vanquish the hound: to compile Guile into fast bytecode. There would be no inefficient association-lists of bindings at run-time. Indeed, there would be no "environment object" to capture. I succeeded, and with Guile 2.0, `local-eval`, `procedure->syntax` and `the-environment` were [no more](http://git.savannah.gnu.org/cgit/guile.git/tree/NEWS?h=stable-2.0&id=7e9a301b7f3bcc811803305250b22d71a8b06155#n924).

But no. As Guile releases started to make it into distributions, and users started to update their code, there arose such a howling on the mailing lists as set my hair on end. The ghost of `local-eval` was calling: it would not be laid to rest.

I resisted fate, for as long as I could do so in good conscience. In the end, Guile hacker Mark Weaver led an expedition to the mailing list moor, and came back with a plan.

Mark's plan was to have the syntax expander recognize `the-environment`, and residualize a form that would capture the identities of all lexical bindings. Like this:

```
(let ((x 10)) (the-environment))
=>
(let ((x 10))
  (make-lexical-environment
   ;; Procedure to wrap captured environment around
   ;; an expression
   wrapper
   ;; Captured variables: only "x" in this case
   (list (capture x))))

```

I'm taking it a little slow because hey, this is some tricky macrology. Let's look at `(capture x)` first. How do you capture a variable? In Scheme, with a closure. Like this:

```
;; Capture a variable with a closure.
;;
(define-syntax-rule (capture var)
  (case-lambda
    ;; When called with no arguments, return the value
    ;; of VAR.
    (() var)
    ;; When called with one argument, set the VAR to the
    ;; new value.
    ((new-val) (set! var new-val))))

```

The trickier part is reinstating the environment, so that `x` in a local-eval'd expression results in the invocation of a closure. [Identifier syntax](http://www.gnu.org/software/guile/manual/html_node/Identifier-Macros.html#index-identifier_002dsyntax-1899) to the rescue:

```
;; The wrapper from above: a procedure that wraps
;; an expression in a lexical environment containing x.
;;
(lambda (exp)
  #`(lambda (x*) ; x* is a fresh temporary var
      (let-syntax ((x (identifier-syntax
                        (_ (x*))
                        ((set! _ val) (x* val)))))
        #,exp)))

```

By now it's clear what `local-eval` does: it wraps an expression, using the wrapper procedure from the environment object, evaluates that expression, then calls the resulting procedure with the case-lambda closures that captured the lexical variable.

So it's a bit intricate and nasty in some places, but hey, it finally tames the ghostly hound with modern Scheme. We were able to build [`local-eval`](http://www.gnu.org/software/guile/manual/html_node/Local-Evaluation.html) on top of Guile's procedural macros, once a couple of accessors were added to our expander to return the set of bound identifiers visible in an expression, and to query whether those bindings were regular lexicals, or macros, or pattern variables, or whatever.

**"watson, your service revolver, please."**

As that Guile discussion was winding down, I started to hear the howls from an unexpected quarter: JavaScript. You might have heard, perhaps, that [JavaScript eval is crazy](//wingolog.org/archives/2012/01/12/javascript-eval-considered-crazy). Well, it is. But ES5 strict was meant to kill off its most egregious aspect, in which eval can introduce new local variables to a function.

Now I've been slowly hacking on implementing block-scoped `let` and `const` in JavaScriptCore, so that we can consider switching [gnome-shell](http://live.gnome.org/GnomeShell) over to use JSC. Beyond standard ES5 supported in JSC, existing gnome-shell code uses `let`, `const`, destructuring binding, and modules, all of which are bound to be standardized in the upcoming ES6. So, off to the hack.

My initial approach was to produce a correct implementation, and then make it fast. But the JSC maintainers, inspired by the idea that " `let` is the new `var`", wanted to ensure that `let` was fast from the beginning, so that it doesn't get a bad name with developers. OK, fair enough!

Beyond that, though, it looks like TC39 folk are eager to get `let` and `const` into all parts of JavaScript, not just strict mode. Do you hear the hound? It rides again! Now we have to figure out how block scope interacts with non-strict eval. Awooooo!

Thankfully, there seems to be a developing consensus that `eval("let x = 20")` will *not* introduce a new block-scoped lexical. So, down boy. The hound is at bay, for now.

**life with dogs**

I'm making my peace with eval. Certainly in JavaScript it's quite a burden for an implementor, but the current ES6 drafts don't look like they're making the problem worse. And in Scheme, I'm very happy to provide the [primitives](http://www.gnu.org/software/guile/manual/html_node/Syntax-Transformer-Helpers.html) needed so that local-eval can be implemented in terms of our existing machinery, without needing symbol tables at runtime. But if you are making a new language, as you value your life, don't go walking on the local-eval moors at night!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [value representation in javascript implementations](/archives/2011/05/18/value-representation-in-javascript-implementations)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)

### 3 responses

1. Jasper St. Pierre says:[2 February 2012 6:11 AM](#a8760f885ea21ab8e63141c4356c913736dbcc82)

   "Now I've been slowly hacking on implementing block-scoped let and const in JavaScriptCore, so that we can consider switching gnome-shell over to use JSC. Beyond standard ES5 supported in JSC, existing gnome-shell code uses let, const, destructuring binding, and modules, all of which are bound to be standardized in the upcoming ES6. So, off to the hack."

   gnome-shell will never be ported to JSC. Besides the features that JSC is missing, we're also relying on GJS, which implements a hell of a lot of stuff and is pretty tied around the Mozilla JS API, and we're adding more every day. In fact, we're going to be landing a patch soon that introduces meta-classes into GJS as part of implementing GObject inheritance. The problem is that metaclasses rely on \_\_proto\_\_ to work properly, so that we can have a custom \[\[Prototype\]\] on a constructor, which, as far as I can tell, isn't allowed through regular ECMAScript 6 -- we need constructors to be both subclasses and instances of the base metaclass.

   That's just one example. If you really want to port GJS to JSC, be my guest, but you're going to have some trouble. We're not going to port GNOME Shell to Seed -- we need one JavaScript runtime in GNOME, and at the moment, GJS is the more advanced one, maybe because GNOME Shell and GNOME Documents use it.

2. [Marijn Haverbeke](http://marijnhaverbeke.nl) says:[2 February 2012 9:39 AM](#10019b822e260b2a30701ab6b57b7be7fa104b98)

   Interestingly, this is quite similar to what CL-JavaScript does to support in-scope eval without maintaining its own environment. See https://github.com/akapav/js/blob/0a351b08cff2dc4b2e9979eaec4a88e36b5203d3/translate.lisp#L72

3. Brandon says:[17 February 2012 11:48 PM](#3a920f19b2eb5cda58a19e89f89a7aa868565c07)

   I see you've been reading a certain classic English fiction author again...

Comments are closed.
