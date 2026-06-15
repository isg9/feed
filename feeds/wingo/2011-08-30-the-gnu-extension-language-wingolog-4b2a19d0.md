---
title: the gnu extension language — wingolog
url: https://wingolog.org/archives/2011/08/30/the-gnu-extension-language
published: "2011-08-30T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/08/30/the-gnu-extension-language
---

# the gnu extension language — wingolog

## [the gnu extension language](/archives/2011/08/30/the-gnu-extension-language)

30 August 2011 6:15 PM

- [gnu](/tags/gnu)
- [guile](/tags/guile)
- [scheme](/tags/scheme)

I'm sitting on a train waiting to leave Paris for Barcelona, returning from the [2011 GNU Hackers Meeting](http://www.gnu.org/ghm/2011/paris/). It was fantastic, clearly the best we have had yet. Thanks a lot to Ludovic Courtès for organizing it, and to [IRILL](http://www.irill.org/) and Sylvestre Ledru for hosting. I hope to write more about it in the future, but this essay will be long enough :)

I gave a talk entitled "The User in the Loop", which made the perhaps obvious argument that "extensibility is good & stuff". I hope that it did so in an entertaining and illuminating fashion. It also argued that Guile is a great fit for the extensibility needs of the GNU project. The video will be out shortly. Slides are available [here](http://www.gnu.org/ghm/2011/paris/slides/andy-wingo-guile.pdf), though you probably just want the [notes](http://www.gnu.org/ghm/2011/paris/slides/andy-wingo-guile-notes.pdf) instead.

**going meta: goals and strategies**

Guile is the GNU extension language. This is the case because Richard Stallman said so, 17 years ago. Beyond being a smart guy, Richard is powerfully eloquent: his "let there be Guile" proclamation was sufficient to activate the existing efforts to give GNU a good extension language. These disparate efforts became a piece of software, a community of hackers and users, and an idea in peoples' heads, on their lips, and at their fingertips.

So, Guile is a great language. But is it still the right extension language for GNU? At this GHM, Jim Blandy brought up the good point that we should revisit our past decisions periodically, to see if they still fit with our goals and with the world. " *X* is the GNU *Y*" is still a powerful, generative proclamation, so we should be careful with that power, to make sure that we are generating the right world. In that spirit, I would like to re-make the argument for Guile as the GNU extension language.

**goal**

Why care about extension languages? My presentation deals with this issue at length, but a nice summary can be found in the Guile manual:

> Guile was conceived by the GNU Project following the fantastic success of Emacs Lisp as an extension language within Emacs. Just as Emacs Lisp allowed complete and unanticipated applications to be written within the Emacs environment, the idea was that Guile should do the same for other GNU Project applications. This remains true today.
>
> The idea of extensibility is closely related to the GNU project's primary goal, that of promoting software freedom. Software freedom means that people receiving a software package can modify or enhance it to their own desires, including in ways that may not have occurred at all to the software's original developers. For programs written in a compiled language like C, this freedom covers modifying and rebuilding the C code; but if the program also provides an extension language, that is usually a much friendlier and lower-barrier-of-entry way for the user to start making their own changes.
>
> \-\- [*Guile and the GNU Project*](http://www.gnu.org/software/guile/manual/html_node/Guile-and-the-GNU-Project.html)

Although GNU maintainers are free to do as they wish with their packages, we are working on making an integrated system, and a system that mostly uses one extension language is better than one with many implementations. It is reasonable to promote one language as the default choice, while not forbidding others, of course.

I think that Guile is a great strategy to achieve this goal. Guile is good for the programs it extends, it is good for users, and it is good for GNU as a whole. The rest of this article argues these points in more detail.

**languages do matter**

An extension language is first and foremost a language: a vehicle for expression and for computation. Guile is an implementation of Scheme, one of the most expressive languages out there.

In his 2002 essay, [Revenge of the Nerds](http://www.paulgraham.com/icad.html), Paul Graham argues (somewhat polemically) that some languages are better than others, and that you should use the better languages. Of course he identifies "better" with "closer to Common Lisp", which is somewhat naïve, but the general point still stands.

At the time when Guile was started, it was quite clear that Scheme was indeed more expressive than Bourne shell, or than Tcl. Since then, many languages have come and gone, and most of those that have stuck around do have many of the features of Scheme: garbage collection, closures, lexical scope, etc.

But still, there are many areas in which Guile Scheme remains more powerful than other languages.

**macros: language extensibility**

First and foremost, Scheme's macros have no comparison in any other language. Macros let users extend their language. Macros allow a language be adapt to time, to the different needs of the future (unevenly distributed as it is).

I realize that this broad argument will not speak to people that do not use macros, so here is a concrete example.

Some languages have pattern-matching facilities, like Erlang. Pattern-matching goes like this: you have a datum, and a set of expected patterns. You go over the patterns in order, seeing if the datum matches the pattern. If it matches, some matching part of the datum is extracted and bound to local variables, and a consequent expression is evaluated.

With most languages, either you have pattern matching, because Joe Armstrong put it there, or you don't, in which case you are either blissfully ignorant or terminally frustrated. But in languages with macros, like Scheme, you can extend your programming language to give it pattern-matching features.

I'll show an implementation here, to keep things concrete. Here's a definition of a `match` macro. If you are unfamiliar with Scheme macros, do take a look at the relevant [section in Guile's manual](http://www.gnu.org/software/guile/manual/html_node/Macros.html).

```
(define-syntax match
  (syntax-rules (else)
    ;; If we get to an `else' clause, evaluate its body.
    ((match v (else e0 e ...))
     (begin e0 e ...))
    ;; Use `pat' helper macro to see if a clause matches.
    ((match v (p e0 e ...) cs ...)
     (let ((fk (lambda () (match v cs ...))))
       (pat v p (begin e0 e ...) (fk))))))

```

There are two rules in this macro. The first rule hands off the real work to a helper macro, `pat`. The second just matches an `else` clause at the end.

```
(define-syntax pat
  (syntax-rules (_ unquote)
    ;; () succeeds if the datum is null, and fails
    ;;  otherwise.
    ((pat v () kt kf)
     (if (null? v) kt kf))
    ;; _ matches anything and succeeds.
    ((pat v _ kt kf)
     kt)
    ;; ,VAR binds VAR to the datum and succeeds.
    ((pat v (unquote var) kt kf)
     (let ((var v)) kt))
    ;; If the pattern is a pair, succeed if the datum
    ;; is a pair and its car and cdr match.
    ((pat v (x . y) kt kf)
     (if (pair? v)
         (let ((vx (car v)) (vy (cdr v)))
           (pat vx x (pat vy y kt kf) kf))
         kf))
    ;; Literals in a pattern match themselves.
    ((pat v lit kt kf)
     (if (equal? v (quote lit)) kt kf))))

```

The `pat` helper macro is written in so-called continuation-passing style. This means that `pat` receives the consequent expressions to evaluate as arguments: one if the pattern matches ( `kt`), and the other if it doesn't ( `kf`). I have commented the clauses to help readers that are unfamiliar with `syntax-rules`.

I hope that you find this implementation to be elegant, but that's not really the important thing. The important thing is that if we put this code in a module and export `match`, then you have provided pattern matching to a language that didn't have it. You, as a user, have extended your programming language.

This facility is quite useful. Let us consider the case of some HTML, which you have parsed to an SXML representation:

```
(define html '(a (@ (href "http://gnu.org/")) "GNU"))

```

We can use the `match` form to get the URL for this link:

```
(define (link-target x)
  (match x
    ((a (@ (href ,target)) . _) target)
    (else #f)))

(link-target html) => "http://gnu.org/"
(link-target '(p "not a link")) => #f

```

Sometimes people say that macros make code hard to read, because (they say) everyone makes up their own language. I hope that this example demonstrates that this is not the case: `link-target` above is eminently more readable, robust, and efficient than rolling your own matcher by hand. Good programmers use macros to bring the language up to the level of the problem they are solving.

Note that you didn't need to build consensus in the ECMAScript committees to add `match` to your language. Nor do you have to be satisfied with Joe Armstrong's particular implementation of matching, as you do in Erlang. [Guile's default matcher](http://www.gnu.org/software/guile/manual/html_node/Pattern-Matching.html) has a number of bells and whistles, but you are free to write custom matchers for other formats, extending matchers to protocols and binary records of various types.

Finally, this same argument applies to embedded parsers, domain-specific languages, dynamic binding, and a whole host of other things. You don't have to wait for a language like Python to finally add a [`with` statement](http://www.python.org/dev/peps/pep-0343/); you can add it yourself. To paraphrase Alan Perlis, Guile is an extensible extension language.

**delimited continuations**

[Delimited continuations](http://www.gnu.org/software/guile/manual/html_node/Prompts.html) let a user extend their code with novel, expressive control structures that compose well with existing code.

I'm going to give another extended example. I realize that there is a risk of losing you in the details, dear reader, but the things to keep in mind are: (1) most other programming languages don't let you do what I'm about to show you; and (2) this has seriously interesting practical applications.

So, let's take coroutines as the concrete example. Guile doesn't define a standard coroutine abstraction yet, though it probably should. Here is one implementation. We begin by defining `call-with-yield`:

```
(define (call-with-yield proc)
  (let ((tag (make-prompt-tag)))
    (define (handler k . args)
      (define (resume . args)
        (call-with-prompt tag
                          (lambda () (apply k args))
                          handler))
      (apply values resume args))

    (call-with-prompt
     tag
     (lambda ()
       (let ((yield (lambda args
                      (apply abort-to-prompt tag args))))
         (proc yield)))
     handler)))

```

The guts of this function is the `call-with-prompt` call, which makes a `yield` procedure, and passes it to `proc`. If the `yield` is called, it will jump back up the stack, passing control to `handler`.

The handler builds a `resume` function, and returns it along with the values that were yielded. The user can then call the `resume` function to continue computation, possibly passing it some values.

If we would like to add some syntactic sugar to make this form easier to read, we can do so:

```
(define-syntax with-yield
  (syntax-rules ()
    ((_ yield exp exp* ...)
     (call-with-yield
      (lambda (yield) exp exp* ...)))))

```

As an example, here is a function that turns a traversal operator into a generator:

```
(define (generate traverse collection)
  (with-yield yield
    (traverse yield collection)))

```

Let's see a use of it.

```
> (generate for-each '(three blind mice))
$1 = #
$2 = three
> ($1)
$3 = #
$4 = blind
> ($3)
$5 = #
$6 = mice
> ($1)
$7 = #
$8 = blind
> ($5)
>

```

This is fascinating stuff. Normally when you call `for-each`, you lose control of your execution. `for-each` is going to call your function as many times as it wants, and that's that. But here we have [inverted control-flow](//wingolog.org/archives/2005/04/06/100) so that we're back in charge again.

Note that we did this without any cooperation from `for-each`! In Python, if you want to compose a number of its generators into a coroutine, every procedure in the call stack must cooperate. In fact until recently this wasn't even possible to implement efficiently. But in Guile, delimited continuations allow us to remove the unnecessary restrictions that make [`yield from`](http://www.python.org/dev/peps/pep-0380/) seem like a good idea. Our `with-yield` *composes* with the other parts of our language.

Also note that these generators are functional: instead of mutating an object, causing the next call to it to proceed, we return the `resume` procedure as a value. It's not always what you want, and indeed you could make different choices, but this does have the advantage that you can rewind your traversal to different points, as I did above by calling `$1` after calling `$3`. You can use this strategy to implement other advanced control-flow operators, like [`amb`](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-28.html#%_sec_4.3.1).

One can use this approach in other inversion-of-control contexts. Node.js people are all excited about their asynchronous idioms, but they have no idea how much better it could be if JavaScript actually supported asynchronous idioms on a language level.

For more on delimited continuations, see the readable 1993 paper by Dorai Sitaram, [Handling Control](http://www.ccs.neu.edu/scheme/pubs/pldi93-sitaram.pdf).

**the next 700 language features**

Guile has a number of other language features that other extension languages do not. Here's a short list.

[goops](http://www.gnu.org/software/guile/manual/html_node/GOOPS.html)

GOOPS is Guile's Object-Oriented Programming System. It implements OOP like Common Lisp's CLOS does.

What this means to most programmers is that GOOPS is a little different. In Smalltalk-style OOP, the focus is on classes. In Self-style OOP, the focus is on prototypes. But in CLOS-style OOP, the focus is on generic functions. A good introduction to this topic can be found in Gregor Kiczales' beautiful book, [The Art of the Metaobject Protocol](http://mitpress.mit.edu/catalog/item/default.asp?ttype=2&tid=3925). At OOPSLA 1997, Alan Kay called it "the best book written in computing in ten years".

Anyway, what I mean to say here is Guile does support object-oriented programming, and although it is not the same as (for example) Python's model, it is quite capable and expressive. Some of its features, like [class redefinition](//wingolog.org/archives/2009/11/09/class-redefinition-in-guile), are not found in any of the more common OO systems.

parallel computation

Guile supports POSIX threads, with no global interpreter lock. Coordination between threads is managed via the normal primitives like mutexes and conditional variables, though there are higher-level forms like `with-mutex`.

Pthreads provide concurrency. Higher-level constructs provide structured parallelism, like [futures](http://www.gnu.org/software/guile/manual/html_node/Futures.html), for fine-grained parallelism, and other forms like [par-map](http://www.gnu.org/software/guile/manual/html_node/Parallel-Forms.html).

I'm going to go out on a limb here and say that managing concurrency is *the* major problem of programming today. Guile's existing facilities are OK, but are not as convincing as those of Clojure or D. Is STM the right answer? Or is it shared-nothing by default, like Erlang? I don't know, but I'm pretty sure that we can build the right answer in Guile, whatever the right answer is.

the numeric tower

Guile implements the full numeric tower: integers (of any precision), real numbers, rationals, complex numbers, and the strange IEEE-754 values like negative zero, not-a-number, and the infinities. Arithmetic works with them all. In addition, there are some plans afoot to add support for optional multi-precision floating-point and complex numbers.

I shouldn't have to say this, but in Guile, `(/ 12 10)` is not `1`. Nor is it a double-precision approximation, for that matter; it is the rational number, `6/5`.

data types

Guile comes with a wealth of built-in data types: bytevectors, uniform arrays (for example, a vector of packed unsigned 16-bit integers), multidimensional arrays (possibly of packed values), bitvectors, strings consisting of characters (as opposed to bytes), records, etc. There are also the standard Lisp data types like symbols, pairs, vectors, characters, and so on.

You can define your own data types, using GOOPS, using the record facility, or the lower-level structs. Of course you can build ad-hoc data structures using the other predefined data types; after all, as Alan Perlis says, "it is better to have 100 functions operate on one data structure than 10 functions on 10 data structures."

functional programming

Scheme is a multi-paradigm language, supporting many kinds of programming techniques. It has excellent support for the functional style, with its lexical scope, closures, and proper tail calls. Also, unlike their `call/cc` cousins, Guile's delimited continuations are practical to use in a functional style, as shown above in the generator example.

Finally, I should mention that all of these features are designed to work well together. Macros compose with modules to preserve lexical scoping. Delimited continuations can transfer control through functional combinators while preserving referential transparency. Dynamic bindings behave sanely when continuations are unwound and reinstated. Scheme is a language that "hangs together": its features are carefully chosen to produce sound semantics when used in combination.

**implementation features**

So much for the Scheme language, as implemented by Guile. On a related but not identical tack, I would like to argue that Guile's implementation is a good practical fit for GNU.

development experience

Guile has a fantastic developer experience. Two factors contribute to this. One is that it ships with a [full-featured, pleasant REPL](http://www.gnu.org/software/guile/manual/html_node/Using-Guile-Interactively.html), with tab-completion, readline, and a debugger. The second factor is Guile's reflective capabilities, coupled with the excellent [Geiser](http://www.nongnu.org/geiser/) Emacs mode.

As Geiser hacker [Jao](http://programming-musings.org/) says, when it comes to debugging, you can be a mortician or you can be a physician. Core dumps are corpses. The REPL is the land of the living. If you run Guile with the `--listen` argument, it will spawn off a REPL thread, listening on a port. Geiser can telnet into that port to work with your program as it is running.

See [Using Guile in Emacs](http://www.gnu.org/software/guile/manual/html_node/Using-Guile-in-Emacs.html#Using-Guile-in-Emacs), for more on how to hook this up.

multiple languages

It's pretty clear to me that we're going to nail the Elisp / Scheme integration. BT Templeton has been doing some great work on that this summer, and we are closer than ever. Admittedly some folks have doubts about how we will pull this off, but I am convinced. I will have to defer detailed arguments to some other forum, though.

Now, when Richard first promoted Guile as the GNU extension language, I think it was because he loved Lisp, but he did consider that not every user would share this perspective. For that reason he stressed the multi-lingual characteristics of Guile, which until now have not really come to fruition. However it does seem that in addition to Scheme and Emacs, with time we can probably get Lua and JavaScript working well, to the point that users will feel happy making small- and medium-sized programs and extensions in these languages.

It's true that if we decided that our best strategy would be to make JavaScript hackers happy, then we should adopt V8 instead, or something like that. But I don't think that is our best strategy. I will come back to this point in the conclusion.

health

Guile is some 16 years old now, and time has winnowed its interface to something that's mostly clean and orthogonal. There are still some crufty pieces, but we have a good [deprecation story](http://www.gnu.org/software/guile/manual/html_node/Deprecation.html), both for Scheme code and for C code linking to `libguile`.

Despite its maturity, Guile is a very healthy project. Check the [Ohloh page](https://www.ohloh.net/p/guile) for details, but note that the activity statistics are based on the `master` branch, whereas most commits have been landing on `stable-2.0` since 2.0.0 was released in February. We'll move back into a development phase at some point in the next few months.

And on the level of users, things seem to be going well -- it's not the most-used language out there, but it is gaining traction and in respectability. The number of contributors is going up, and the various forums are active.

**choosing a strategy to achieve our goals**

I have argued, I hope convincingly, that Guile is a good choice for an extension language. Now I would like to put the decision itself in a context. As Noam Chomsky keeps reminding us, we need to judge decisions by their expected consequences, not their proclaimed intentions. Does Guile generate the right future for GNU?

Let me begin by arguing this point from the contrapositive: that choosing another language is *not* the right thing for GNU. Let us take the example of JavaScript, a very popular language these days. Jim Blandy notes that the real measure of an extension language is that, when you want to implement a feature, that you want to implement it in the extension language rather than the original language (C, usually). So let us assume that JavaScript triumphs in this regard in the future, that the GNU system ends up mostly implemented in JavaScript, perhaps via the V8 implementation. Is this the right future to generate?

For me, the answer is *no*. First of all, what does it mean to have a GNU system based on a non-GNU technology? What would be left of GNU? It might be lack of imagination on my part, but such a future makes me think of country folk come to the city and losing sight of who they are. GNU is a social organization that makes software, with particular social characteristics. Mixing GNU with a foreign language community sounds dangerous to GNU, to me. See Kent Pitman's [Lambda, the Ultimate Political Party](http://www.nhplace.com/kent/PS/Lambda.html) for similar musings.

Beyond the cultural issues -- which to me are the [ones that really matter](//wingolog.org/archives/2008/07/10/how-to-choose-between-equivalent-options) \-\- there are some important technical points to consider. Right now GNU is mostly written in C, and controls its implementation of C, and the libraries that it depends on. Would GNU be able to exercise such control on JavaScript? Can GNU build in new concurrency primitives? Could we add new numeric types? Probably not. JavaScript is a language with a platform, and that platform is the browser. Its needs are not very related to the rest of GNU software. And if we did adapt JavaScript to POSIX-like platforms, GNU would lose some of the advantages of choosing a popular language, by making a second-class, foreign platform.

We should also consider the costs of using hastily designed languages. JavaScript has some crazy bad stuff, like `with`, `var` hoisting, a poor numeric model, dynamic `this` scoping, lack of modularity regarding binding lookup (witness the recent spate of browser bugs regarding prototype lookup on user objects), the strange `arguments` object, and the lack of tail recursion. Then there are the useful features that JavaScript lacks, like macros, modules, and delimited continuations. Using JavaScript has a cost, and its total cost in GNU would have to be integrated over its lifespan, probably 20 years or so. These limitations might be acceptable now, but not in the context of the larger programs of the future. (This, by the way, was the essence of Richard's argument against Tcl.)

Finally, we have the lifespan issue. If GNU had chosen Tcl because it was popular, we would have a mass of dead code. (You can like Tcl and still admit that we are past its prime.) Python is now, I think, at the height of its adoption curve, and beginning its descent. JavaScript is the next thing, and still on the uptick. But JavaScript will fall out of favor, and there will be a new paradigm, and a next language. The future of computing will not be the same as the present. So how will JavaScript adapt to these changes? We can't tell right now, but given the difficulty in changing simple things like making `010` parse as 10 and not 8 indicates that at some point it will stagnate. But Scheme will still be with us, because its parts are well thought-out, and because it is a language that is inherently more adaptable.

Paul Graham concludes [Revenge of the Nerds](http://www.paulgraham.com/icad.html) with the following advice:

> "If you start a startup, don't design your product to please VCs or potential acquirers. *Design your product to please the users.* If you win the users, everything else will follow. And if you don't, no one will care how comfortingly orthodox your technology choices were."

In the case of a GNU extension language, our users are developers, so we do need to work hard to please them. But free software has a different life cycle than a VC-backed startup; instead of racing to the flip, we have to please our users now *and* in ten years. Viewed from the long now, the future is not simply a first-order extrapolation of the present: it is something that we generate. Guile is still the right choice for the GNU extension language because it generates the right future: a healthy, sustainable, powerful GNU project that adapts to the changing needs of time.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

### 37 responses

1. apgarcia says:[30 August 2011 11:06 PM](#96f2ccf47f39ec2c1ed3559394756ce8dbff9d56)

   it seems like all the "cool kids" are scripting in ruby.

   iirc, rms also proposed an imperative language for extensibility that never materialized. scheme is pure (if not purely functional), it is beautiful, ideal, but...you already know the drawbacks.

   thank you for this fine apologetic. :-)

2. Robert says:[30 August 2011 11:55 PM](#32b5108fbe4bbe3991ac4902696164d2dd42b609)

   It isn't whether Guile is good, bad, usable, or not usable. It is whether people will and do USE it. I have not seen that as the case for Guile or Scheme in general.

3. cesar says:[31 August 2011 0:17 AM](#3229fcf14ef38d6d9f9aeb830e315c4d780c6b72)

   I really like Scheme and Lisp in general, but why not consider Javascript instead of Scheme? If the idea is to please the developers, Javascript has proven to please them and it is just gonna get better, plus is extremely popular.

4. [Janne](http://janneinosaka.blogspot.com) says:[31 August 2011 0:49 AM](#c19524224adbc0f953b38987a753829aee6cf71c)

   I like the idea of Scheme, and I've long sort of looked for a reason to learn it properly. I'm sympathetic.

   But you just made an eloquent argument against adopting Guile or Scheme as a general extension language.

   For all their cruft and design oddities, Javascript, Python, Ruby, shellscript - even Erlang to some extent - are basically readable by somebody with general programming experience even if they've never encountered that particular language before. A bit of thinking and a few net searches later and you can even do minor tweaks to adapt a script to your needs with a fair chance of success.

   But what you wrote above is absolutely incomprehensible. I look at that pattern matching code and I have no clue what is going on. Neither will most other people encountering it.

   Yes, if I spend a couple of days or weeks reading up on the language properly, play with examples and so on I expect I will gradually be able to read - and more importantly, be able to tweak - that code. But I would need a fairly serious time investment to do so.

   That kind of time investment is fine for a "deep" language, one that you expect to use as a primary language for your work. I've spent that kind of time and more learning Python, C++ and other languages after all.

   But an extension language invites quick hacks and simple tweaks of existing scripts, and should make that easy even for the novice. It should be a language that empowers the masses of people of little skill and indifferent talent, even if it hurts the few of prodigious skill and deep experience. Scheme, unfortunately, does not.

   I don't know what extension language GNU should use. I don't even know if adopting a single language is the right way to go (M4, bash and others aren't going away after all). I do know that for it to have any chance of success it has to be a language in the vein of Perl/JS/Python/Ruby/Lua and so on, and be accessible to people in much the same way.

5. [Grant Rettke](http://www.wisdomandwonder.com) says:[31 August 2011 3:24 AM](#4ca968f428c3983c565469f5af1c0f14ad2bc139)

   Great article Andy and great work Guile team!

6. Kettle Pot says:[31 August 2011 5:42 AM](#6090f28b7281fd05b27964997810f2f96a06cf9c)

   Guile looks like it was designed by monkeys. Seriously, who would bother with this rubbish when there are good solid very popular scripting languages such as Javascript, VBScript and Forth?

7. Denis Washington says:[31 August 2011 6:27 AM](#5f15c8030f7f925f7376b214780f5025b1e14086)

   @Janne: The code that Andy wrote here is indeed pretty hard to read, but this is macro code - that is, the kind of code that is really the hardest to write even for Scheme programmers, and that you will most probably \*never\* have to write as an program extension programmer. Actual "normal" Scheme is much simpler to grok. (Andy, maybe you should tweak the article to include more "average joe" Scheme, or write a new one with that intention?)

   As to tweakability, I think the extreme success of Emacs Lisp and the wealth of extensions available there speak for themselves: Lisp \*is\* a tweakable language. But again, the code in this post is not what you would encounter in normal extension code; it it pretty advanced.

8. [Andy Wingo](http://wingolog.org/) says:[31 August 2011 7:01 AM](#a61b20b1624c75b6ae080c617598796b8b2be686)

   You are probably aware of this, apgarcia, but you can do imperative programming in Scheme. Scheme is more pragmatic than it is given credit for ;-)

   Cesar, though I was a bit harsh on JavaScript, its acceptance does at least prove the point that people are fine with ending their expressions with [`}}};}());`](http://javascript.crockford.com/code.html)! Compared with that, I think that Scheme's much-maligned parentheses are a blessing.

   Thank you for your thoughtful comment, Janne. That macro definition is indeed quite pithy, and not something that a casual user would write. My argument there was more towards the extensible power of the language. I do think that Scheme is intuitive, as much as programming languages can be, but I think I'd have to write a different article to argue that.

   Fortunately, you don't have to understand the `match` implementation in order to use it; it's as easy as saying, "match *x* against `(a (@ (href ,target)) . _)`. If it matches, return *target*, otherwise `#f`." `link-target` literally reads like that!

   That said, if you are interested, here is an expansion of `(match x (a 1) (b 2) (else 3))`:

   ```
   (let ((fk1 (lambda ()
                    (let ((fk2 (lambda () (begin 3))))
                      (if (equal? x 'b)
                          2
                          (fk2))
     (if (equal? x 'a)
         (begin 1)
         (fk1)))

   ```

   The lambdas are there to make sure that lexical variables bound in previous clauses that didn't match are not in the scope of subsequent match clauses. The compiler will "contify" them, treating them as basic blocks instead of closures. It's tricky, but like I said, this isn't every-day code.

   Finally, if you are going to troll, that's OK as long as it is amusing. So in that spirit I appreciate Kettle Black's contribution!

9. person says:[31 August 2011 9:09 AM](#ce26f9ea9f3dca78dee3d8edb0f834e69736658f)

   If you want random people with relatively little programming skill to write 10 line scripts for your program than use JavaScript. If you want to empower your users to take your software beyond what you could imagine use a Lisp like Emacs did.

10. [Ole Laursen](http://people.iola.dk/olau/) says:[31 August 2011 11:51 AM](#a9de79291946dbdda8ee842f5a14cc9a24e9dfaf)

    Emacs is cited as a good example. But from what I gather, Emacs Lisp isn't terribly highly regarded. The few snippets of real Emacs Lisp code shipped with Emacs I've studied weren't extremely expressive either and would have taken up less space and been much clearer in Python (or Javascript for that matter). We're talking about code written by seasoned hackers.

    I think it's funny you mention numbers, because Lisp syntax for math is arguably far from what most people consider readable (+ (2 (\* 3 2)) - I don't know if that is solvable with macros.

    person: that's just meaningless rhetorics. Javascript is transforming the web, and is probably a bigger bomb under Microsoft's (and Apple's) proprietary dominance than anything else. I don't know anyone writing internal enterprise apps for ordinary GUIs anymore. This is little by little freeing the world from vendor lock-in.

    What the web and Emacs have in common is the fact that they're easy and fun to extend.

11. [Janne](http://janneinosaka.blogspot.com) says:[31 August 2011 2:00 PM](#bec50895a0bc5de534bfc98186483be07a16c88b)

    Andy, I think Denis has a good idea; make a post or two showing what guile would look like as a scripting language for a hypothetical (or not so hypothetical) application.

    Also, and I hope I don't make anybody too angry, but pattern matching is a particular, special case where I do believe you'd be better off leveraging all the existing knowledge and make something closely resembling a typical regexp matcher even if it isn't particularly "Scheme:ish". Give a variable and a regexp string, and match using that. That particular format is so very popular and so expected you'd be setting people up for a fair amount of needless frustration by not supporting it in addition.

    Oh, FIG Forth was my first high-level language (I use the term advisedly here), and I still read it just fine so I didn't even catch the subtext of Kettle Pots comment. ^\_^

12. Ian Price says:[31 August 2011 2:35 PM](#2f0770fea574d78eec51312335087f09f369c140)

    Ole Laursen,

    Infix is sort of possible through macros, as Marco Maggi \[0\] and others have shown, although to make it "as nice" as a language with it built in you need to be able to modify the reader. On the other hand, Maths isn't where I would go for notational inspiration: strange letters everywhere, subscripts, superscripts, 2-d notation, tons of symbols are overloaded (including parentheses) and more precedence rules than you can shake a stick at.

    Of course, that's the biased view of someone who loves Scheme and has regarded precedence rules with suspicion since I was introduced to them in primary school.

    \[0\] https://github.com/marcomaggi/Infix

13. [Carson Chittom](http://www.wistly.net) says:[31 August 2011 4:00 PM](#4848bda5f46ea638bad53bd9fefd4f80e8d85a76)

    Why does there need to be two: Guile and Emacs Lisp? Why split resources? Why not just declare that "Emacs Lisp is the GNU extension language"? Or, alternatively, that Guile is, and that, as of Emacs 25 (or 26, or whatever), Emacs Lisp will conform to Guile? This would probably be a lot of work for somebody (and no, I'm not volunteering), I know, but I'm genuinely puzzled.

14. Eirik says:[31 August 2011 4:51 PM](#2d346a4da82080eae817bd48f478af4ce2461600)

    I've looked briefly at lisp/scheme as well as

    other "advanced" high level languages like

    haskell several times -- and never been quite

    motivated to learn them. The closest language

    I've become semi-fluent in is

    Standard Meta-Language -- but that is also not

    a language I'd be comfortable implementing a couple

    of dialog boxes exposing application settings held

    in a text file in a simple GUI, or presenting

    a search over a ~10000 message mailbox in.

    I don't really think "the profilation of Emacs

    scripts" is a very good indication of the suitability

    of a lisp-dialect as an extention-language -- I think

    it's a good indication of how many lisp programmers

    use Emacs.

    The fact that so many (new and old) developers have

    been able to implement stuff in javascript is no

    testament to the usability of that language, any more

    than the profilation of Exel-macros is a testament to

    the greatness of the various incarnations of Visual

    Basic.

    Desperation leads to implementation -- never mind what

    tools are available. I think Google Gears is a better

    indication of how \*bad\* javascipt is as language.

    Your text implies two things (maybe not intentionally):

    1) It's easier to customize scheme, than write your own

    language/compiler (This may or may not be true; if your

    project is small lexx/yacc or S/ML might be easier for

    \*most\* programmers -- it would certainly be easier if

    your goal was to design an imperative-style language)

    2) Guile isn't a language, it's a (VM) platform. In other

    words it shares some good characteristics with java, .net

    and LLVM -- there's a possibility of a common runtime with

    various interfaces (languages).

    For me as a user of various GNU (and GNU compatible)

    software systems, it's much more important that I can

    pick up some example code, read it and guess how to

    tweak it -- than it is for me to be able to implement

    my own programming language. I have plenty of tools

    for the latter -- the extention platform would be the

    only (easy) way to leverage pieces of GNU software for

    something else -- like writing a searchable web-mail

    archive with a SMTP, IMAP and web frontend, that also

    serves as discussion board, for instance.

    I think supporting Lua in additon to Guile/Scheme would

    solve a lot of these problems. For most developers not

    used to working with functional languages, Lua would feel

    much more natural -- and it would certainly be more readble

    to most. I still don't understand how (even experienced)

    lisp programmers are able to read their own code, and

    figure out what is going on -- without considerable effort.

    Maybe they can't -- maybe they simply don't.

15. Mike C says:[31 August 2011 5:55 PM](#83ba47b98cd2bd85f886cee763fecd19fd44cd7c)

    When considered abstractly against all languages, guile looks pretty good. But considered against the (arguably) best concrete alternative--Python--guile doesn't look like a very strong contender.

    Yes, Python lacks threading (though PyPy may soon solve that) and probably never will have usable macros. But for an extension language, these seem like extremely weak complaints to stand up against Python's many advantages.

    I think it would be useful to have an essay specifically making the case for Guile over Python.

16. [David A. Wheeler](http://www.dwheeler.com/readable) says:[31 August 2011 7:54 PM](#e702c3ca0af6dfbe7139ef13cce2d974c73c0f66)

    I like Scheme, there's much to like about it. But as comment #4 notes, lots of people find Scheme absolutely unreadable. If you want a language to be widely used as an extension language, it needs to be easy to write and easy to read. You might want to take a lot at my "readable" extensions, e.g, sweet-expressions, here: http://www.dwheeler.com/readable.

    It lets you use Scheme, with macros and all, in a readable form.

17. Bruce says:[31 August 2011 7:59 PM](#c3888008d354479665bf6c3d8c9ef3f98a101091)

    Why the cheap shots against Tcl? Not only is Tcl still alive and thriving, it has specifically succeeded wonderfully well in the role of extension language, namely for the Cisco router operating system, reaching a level of distribution and use thereby that Guile users can ony dream of.

    And why the accusation of "dead code?" Tcl again has been almost uniquely successful among dynamic languages in preserving backward compatibility. Even on the twenty-year time horizon mentioned above, code written decades ago generally still runs without alteration in current interpreters.

    The article reads like you were specifically trying to present an inverted picture of the strengths of Tcl.

18. [Ryan McCoskrie](http://sourcelinksnotes.comyr.com) says:[31 August 2011 8:04 PM](#1bec2e07d0f7dc761fce289bc722388a4f67b8f2)

    One thing that Lisp programmers tend not to realise about themselves is that they are elitist. They think that because they do their best programming in Lisp anyone who doesn't get Lisp isn't really a programmer. That's not true, a programmer is someone who can write a program regardless of whether they have that special mathematical knack needed to deal with Common Lisp, Scheme, Hackell etc.

    And yes, you may technically be able to write imperative code in guile, but can you think imperatively in guile?

19. [Andy Wingo](http://wingolog.org/) says:[31 August 2011 8:32 PM](#a5a4dfc4d3cbf5ea4b33ba802e4eb40433123179)

    Janne, I think I may have not made myself clear; I meant to match against a compound object, not a string. Guile does have regular expressions for working with strings. For example, `(p "not a link")` is the Scheme syntax for a list of two elements, the symbol `p` and the string `"not a link"`.

    David, that's an interesting link. Do you prefer to program with "sweet expressions"? I find that I miss paredit when I hack in C, these days...

    Bruce, I sincerely apologize. I didn't know that this article would see this wide distribution, but that's no excuse; in the language world, we have this (correct) detente that we don't criticize other minority languages, because the network effects of not being the top dog blunt the facilities that each language brings to the table. I think that applies to Scheme as well as to Tcl. I really do apologize for any slight that I gave to the Tcl community.

    That said... it seems, from reading the archives (because I wasn't there) that Tcl really had a practical heyday around 1993 or so. Guile was made around that time, and partly in the context of "OMG Tcl is going to steamroll us". My point is basically that popularity is a fickle thing to depend on, and that to affect the future, you have to build it, starting from wherever you are.

20. [Andy Wingo](http://wingolog.org/) says:[31 August 2011 8:35 PM](#9abd11eb241191ecca92740a9515ccc1e2ce4984)

    Ryan, unfortunately those people do exist, and actually I think they are damaging to the Lisp family of language. However, the existence of elitists doesn't affect the validity of Graham's argument about some languages being more powerful than others.

21. saulgoode says:[1 September 2011 3:03 AM](#f3ab68953af7a3eee47e054cfd2ddc361ab95b76)

    I agree with the technical merit of Guile being considered GNU's default extension language, however, in order to see wider acceptance, it needs to support the ability to create and receive user options from graphical dialogs.

    The graphical support module of Guile needs to be readily available and reliable on GNU platforms and, unfortunately, this has historically not been the case and application developers have had to look elsewhere for their extension languages even when all that is needed is a dialog to pop up and accept user input. Even for those situations where Scheme was chosen as the plug-in language, interpreters other than Guile (SIOD, Nyquist, TinyScheme, or one-off interpreters) are chosen and limited GUI dialog interfaces implemented in C. All too often, other languages such as TCL, Perl, or Python are adopted mainly because of their ability to interact with the user in a graphic environment.

    In order to see wider adoption as an extension language, Guile also needs to provide developers with better information on the creation of library bindings. The tortoise tutorial is all well and good for guidance on interfacing to a few specific library functions, but it doesn't cover the more elaborate practices that a complete and maintainable interface demands. There is quite a bit of documentation on core concepts but it is in some cases outdated and the aspect of a generalized high-level methodology rarely receives more than cursory mention, More importantly, the high-level methodologies proposed tend to approach the issue from the viewpoint of the library itself providing the Guile support.

    What is needed is topical documentation on best practice in implementing an intermediate layer (employing smobs, snarfing, .defs files, or what-have-you) that provides comprehensive bindings to an existing library and can be administrated as a project independent of both Guile and the library being bound (the library project may not be interested in incorporating Guile-specific support code, nor should they have to).

    I am willing to assist in producing such documentation and in contributing to maintenance of graphical interface modules, however, after several years of investigating options such as Sforms, guile-gtk, and now guile-gnome, I am still at sea with regard to what is expected from a project level standpoint. Any guidance offered would be much appreciated (perhaps a conspectus of your work on the guile-gnome module, or a tutorial on a maintainable approach to interfacing to a small library).

22. [dim](http://tapoueh.org/) says:[1 September 2011 9:31 AM](#a4de91789eb3d9ad72bb2c7371f2f1c79a80df2b)

    There's something that I find very important that has been forgotten about here. Emacs and Emacs Lisp are very successful, on the metric of an extension language and the diversity and quantity of extensions to be found, and I think that's because of the relationship it allows to build between the worker and his tools.

    In Emacs, when something is not working the way you want it to, rather than adapt yourself and the way you think and see the problem, you can just as well tweak Emacs so that you can continue thinking your way. Now the tools are supporting you.

    That's the same idea with macros and programming language. There's this nice way to write code that you want to have, because it's the way you think about the problem you're trying to solve. But the language has no built-in way for you to express yourself this way. No problem, write a macro, now get back on solving your problem \*your\* way.

    That's not everyday's code here, but that's a feature I want to have in an extension language, because that's exactly where I want to tweak existing tools so that I'm free to think my very own way and bend the tool until we work together. Not the other way round. Pretty please.

    Let's stop accepting that users should think about their problems only in terms of the specifics of the tools they're given.

23. [Donal Fellows](http://wiki.tcl.tk/73) says:[1 September 2011 2:08 PM](#2c31b1a52a3a4eba601e39e14ba94a79f0f69e3b)

    The big issue with Guile as an extension language is that it's a Lisp with a VM, which makes it very hard to integrate with a larger application that has its own memory management systems. SML, Haskell, Java, and C# all have the same issue, but don't claim to be practical languages to use in that way in the first place. (OK, I admit that there are languages based on some of them which are more suited - Beanshell is an example - but they assume the outer VM environment in the first place.) If Guile is intended as a language to compete against Lua, Perl, Python, Ruby and Tcl (alphabetical order) then the VM issue \*must\* be addressed. Doing things "the other way round" (i.e., extending a core language with bits and pieces from external libraries) is admittedly far easier, but then you're fighting against lots of languages. Why use Guile when you can use a full Scheme implementation?

    Re Tcl specifically, be careful to distinguish between the situation in 1993 (when many of the criticisms leveled at it were indeed valid) and the situation in the past decade or so. In particular, the charges relating to speed and code organization have been addressed for a long time - and without fundamental language changes. (There are some things that might still apply, notably that Tcl is not typed in a "conventional" way, but that's more of a philosophical difference than a problem per se.)

24. Daniel Svensson says:[1 September 2011 5:31 PM](#6cee72c11e35137061ae320d42ff03e5fbe2d19a)

    It's interesting that most JavaScript proponents here doesn't see the limitations in the numeric system they use. Please try to communicate with a WebService that deals with 64bit integers and see how much fun that is. I wouldn't rely on JavaScript for anything other than changing color of text on hover.

25. Andy Tai says:[1 September 2011 6:16 PM](#3139be09eccb84541a550488dbd815c6d4ee9eb5)

    The success of Lua shows there is a need for a small extension language in many enviroments where things like a full numerical tower may be overkill. Guile should have a such a minialist mode to provide the core scheme functionality with easy extensionabiliy to bind to host (C/C++) services...

26. Andy Tai says:[1 September 2011 7:06 PM](#b7e5f1e02f9322de479fa9ede59cac4169a1b6a3)

    Also, as the guile VM supports Javascript, should that be emphasized... all the pro-Javascript arguments above are not reason against Guile...

27. John says:[1 September 2011 9:16 PM](#8b967516a8129a80b53cc96971f4d012859fb3a4)

    \> Beyond the cultural issues -- which to me are the ones that really matter ...

    My impression is that Guile's main problem is a cultural one: it's viewed only as an extension language -- not as one you'd use by itself to write your next app.

    A prospective new user with SICP in-hand looking at the Guile website will assume that it's only useful for embedding into a C application.

    So, this means that you are currently marketing Guile to an extremely small audience: C programmers working on a C program which needs an embedded scripting language, who also happen to like Scheme.

    A much larger audience to market to would be: people who want to write their own apps in Guile from the ground up.

28. rixed says:[2 September 2011 4:39 AM](#56dd0258b9d5afdb2cb40f2b058844e0e716c63f)

    Andy, you forgot to mention one quite important property of guile that makes it stands out as a language: it was specifically designed to be easy to link to an existing C program. The only other language I know of sharing this property is LUA.

    Seriously, it's hillarious to read here and there (ltu for instance) people arging that python, ruby, racket and so on would make better alternative ; these people certainly never tried to extend a C program with these languages !

    Yes I know it's supposed to be better to extend than to embed (ie to add C functions/types to the scripting language than it is to add a scripting language to a C program, but the evolution often proceeds the other way around (with the actual benefit that it makes possible to extend a program with several distinct scripting languages). Anyway, extending is not ideal neither, since ideally we'd have a single language covering in abstraction the range from C to higher level user friendly language, but that's another story.

    Anyway, go ahead guile folks, keep the good work, the faith and motivation! Keep ignoring opinions about readability, parentheses, what would have been a better choice for GNU and so on. That's not the opinions that matter but the existence of killer apps, then we will use what extension language is available in the killer app. The only thing our opinions are good for, is to make us laugh when we look back at them in the future :-)

29. John says:[2 September 2011 5:11 AM](#faef11c1920e79eecd6afa44bbd2524e6fc9dd47)

    Sorry, typo. In comment #27 I wrote:

    "A much larger audience to market to would be: people who want to write their own apps in Guile from the ground up."

    But that should be s/Guile/Scheme/.

30. [Ian Holmes](http://biowiki.org/IanHolmes) says:[2 September 2011 7:21 AM](#39998ddbabc11fca577c5a65b74f5dc4524e3e09)

    Great work, this sort of outreach really helps. I've been using guile to extend my C++ bioinformatics/phylogeny software. We are writing it up at the moment. My guile efforts are still a bit rough around the edges (no fancy macros or continuations yet) but I appreciate the depth of the language, and more generally hope to help promote functional programming in science. Thanks for putting a powerful tool like this in our hands.

31. [Alaric Snell-Pym](http://www.snell-pym.org.uk/alaric/) says:[2 September 2011 1:36 PM](#bb5e86b7812e3eb3b7ada59a472231774baf77b1)

    I've always been a big fan of extension systems in languages - ones based on an embedded language, although not being the only way to do it, are particularly flexible and easy to use!

    Here's a blog post I wrote on the higher-level issues of how to integrate modular extensiblity into applications, which is food for thought for anyone wanting to make their apps extensible:

    http://www.snell-pym.org.uk/archives/2006/09/15/plugin-based-applications/

32. k says:[2 September 2011 2:19 PM](#4f8eded282ff793866274be578be8a392d101dce)

    Your variable names suck.

33. [Mitch Skinner](http://jbrowse.org) says:[26 November 2011 6:46 PM](#12cf78eb70d1d292e93df8d7f89ee3e2baf858c0)

    (I'm just getting here now because LWN linked it today)

    It's true that javascript lacks macros, modules, and delimited continuations. But (as we're now beginning to see) it's possible for JS to evolve, and some of those things can be added to it. Modules are certainly going in relatively soon. The TC39 people have said more than once that they'd like to have macros; they could certainly use help prototyping them in JS.

    TC39 has, in the past, rejected continuations because of the case where script code is called from native code, and the continuation capture machinery can't capture the state of the native stack (Brendan called this the "script-native-script" problem). But I'm not sure if that objection applies to delimited continuations, or if it's only a problem for "regular" (undelimited) continuations.

    Anyway, the point is, JS isn't static. This is especially true in the GNU extension case, where you're not necessarily targeting IE, and you can make decisions about the implementation you use. Then the question is: can you do more good by helping to add the missing features to JS, or by promoting Guile? From a social utility point of view, if

    (total good) = (per-developer good) \* (number of developers)

    then even a small per-developer improvement generates massive change when multiplied by the number of JS developers.

34. LaPingvino says:[1 December 2011 8:59 PM](#92cf04a4c1e7d465d07a619ef089ab8b5ab0732a)

    Mitch, true, but JavaScript as running with Guile beneath has the advantage that you can compile all JavaScript features to Guile, while the opposite is not so much true... So yes, you are right and no, that doesn't change anything in the value of this choise.

35. [Mitch Skinner](http://jbrowse.org) says:[2 December 2011 9:02 PM](#77df290fbe52e4634e61722bc22193e246ab4a27)

    LaPingvino, the "compile JS to Guile" argument sounds more like rhetoric than strategy. What's the story for JS-Guile interop (calling JS from Guile and vice versa)? What does a Guile API look like from JS? How much slower is JS-in-Guile than V8? I'd bet that it's quite a lot slower.

    Show me people that are actually using the JS-in-Guile approach to do stuff, and then the argument will be more convincing. The "Guile as a compile target" part of the story has always been the weakest part.

    You actually might have more success compiling Guile to JS.

36. [Arne Babenhauserheide](http://draketo.de) says:[18 January 2013 12:15 PM](#2892b61ccad3ab550f838ec28dc52a9a697381e9)

    Coming late to your post, I just want to share that I really like the concepts you talk about, but the code examples don’t make such a good point about them.

    And I think the reason are the variable names: Those names don’t show a reader who does not know what you do which purpose they have.

    Example: Is v intended to mean “value” or “variable”?

    Can you make your code read like plain english?

37. Jacob says:[17 February 2013 4:01 PM](#74d217008ef6b99f85610674e217f998580ba5e6)

    It looks I am also posting a reply a couple of years late. hehe. I agree with some other people that the code examples are not a good way to "sell" Guile to potential customers. Granted, Scheme macros and continuations are very powerful features that allow you to accomplish things no one thought was possible in other languages.

    Nonetheless, my personal fascination with Scheme is in the simplicity and elegance of its \_core\_ language. By this, I mean the subset of Scheme that's used in SICP. You get closures, first class functions, higher order functions, lambda expressions, dynamic memory management, and simple lists that give rise to complex data structures. Granted, other programming languages now provide some of the same features. Nonetheless, Scheme is still setting the standard in simplicity, cleanness, and consistency.

    For example, this is one of the very few programming languages where you don't need to worry about operator precedence at all, because the order of evaluation is always stated explicitly. Scheme syntax is so clean and simple that you can write an interpreter for it in 100-200 lines of code (without bison, flex, etc). In fact, this used to be taught to freshmen at Berkeley and MIT. This is why I sometimes feel horrified when I see examples with macros or continuations. Those are advanced features that should probably be left to the programming language developers.

    Speaking of Guile specifically, what I like that the version 2.0 is so wicked fast. Finally, there is a Scheme interpreter that's not running slow as molasses. Some stupid simple tests I have done suggest that it runs faster than Python 3 for comparable tasks.

Comments are closed.
