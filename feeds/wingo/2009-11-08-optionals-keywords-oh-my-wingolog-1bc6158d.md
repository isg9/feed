---
title: optionals, keywords, oh my! — wingolog
url: https://wingolog.org/archives/2009/11/08/optionals-keywords-oh-my
published: "2009-11-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/11/08/optionals-keywords-oh-my
---

# optionals, keywords, oh my! — wingolog

## [optionals, keywords, oh my!](/archives/2009/11/08/optionals-keywords-oh-my)

8 November 2009 12:33 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [optional](/tags/optional)
- [keyword](/tags/keyword)

OK! Where were we?

In my last dispatch, I talked about [case-lambda in guile](//wingolog.org/archives/2009/11/07/case-lambda-in-guile). The gist of it is, let procedures parse their own arguments, and they can do neat stuff like multiple-arity dispatch.

Also, neat stuff like optional and keyword arguments! Consider our `my-write` example from last time:

```
(define my-write
  (case-lambda
    ((obj port) (write obj port))
    ((obj)      (my-write obj (current-output-port)))))

```

It's a little silly, to write it this way. It's not essentially one procedure with two different bodies, it's one procedure with one required argument, and one optional argument. The optional argument defaults to `(current-output-port)`.

So, as you would imagine, there is a better way to express this "design pattern": `lambda*`, and its sugary friend, `define*`.

In this case, we would simply define `my-write` like so:

```
(define* (my-write obj #:optional (port (current-output-port)))
  (write obj port))

```

So nice, so clear. Default values are only evaluated if the argument is missing. (It's a rare Python programmer that's not surprised about Python's behavior in this regard; but I digress.)

**keyword args too**

Optional arguments are good at allowing for concision and extensibility, but code that uses them can be confusing to read. Actually this is a problem with positionally-bound arguments in general.

I like how [Carl Worth](http://cworth.org/) puts it: that [nice prototypes](http://cworth.org/~cworth/papers/guadec_2006/#(9)) can result in [inscrutable code](http://cworth.org/~cworth/papers/guadec_2006/#(10)). His solution in C is to [have function names encode their arities](http://cworth.org/~cworth/papers/guadec_2006/#(6)), but we can do better in Scheme, with keyword arguments.

So let's say we want to add a "detailed" argument to `my-write`. We can add a keyword argument:

```
(define* (my-write obj
                   #:optional (port (current-output-port))
                   #:key (detailed? #f))
  (if detailed?
      (format port "Object ~s of type ~s" obj (class-of obj))
      (write obj port)))

```

Invocations are really nice to read:

```
(my-write 'foo #:detailed? #t)
=| Object foo of type #<  8c4fca8>

(my-write 'foo (open-output-file "foo.log") #:detailed? #t)
; writes the same thing to foo.log

```

The second example gives an explicit port; and indeed, I am left wondering what it is, when I read it. Keyword arguments make for more readable code.

Keyword arguments also allow for better extensibility. But don't take it from me, take it from P. Griddy:

> Most of the operators in Rtml were designed to take keyword parameters, and what a help that turned out to be. If I wanted to add another dimension to the behavior of one of the operators, I could just add a new keyword parameter, and everyone’s existing templates would continue to work. A few of the Rtml operators didn’t take keyword parameters, because I didn’t think I’d ever need to change them, and almost every one I ended up kicking myself about later. If I could go back and start over from scratch, one of the things I’d change would be that I’d make every Rtml operator take keyword parameters.
>
> \-\- Paul Graham, from a [talk](http://lib.store.yahoo.net/lib/paulgraham/bbnexcerpts.txt) he gave back when he didn't talk about startups so durn much

And, there's one more thing, which applies both to optional and keyword arguments: the default values are evaluated in the lexical context of their preceding arguments. So you can have a later argument referring to an earlier one. For example, Guile's `compile` is defined like this:

```
(define* (compile x #:key
                  (from (current-language))
                  (to 'value)
                  (env (default-environment from))
                  (opts '()))
  ;; wizardly things here
  ...)

```

See how *env*'s default value references *from*? Awesome, yes? I thought so.

**newness**

So what's new about all this? Not much, semantically. Guile has supported `lambda*` and `define*` for more than 10 years. But now they are available in the default environment, and they are fast fast fast -- for the same reasons that case-lambda is faster now. There are special opcodes to process stack arguments into optionals, and to shuffle and bind keyword arguments, all without consing a single cell.

Also, now the toolchain knows about optional and keyword arguments, so that backtraces and printouts show them nicely. For example, `my-write` prints like this:

```
#

```

Ah, there is `case-lambda*`; though it is of dubious utility, given that it can only reasonably dispatch on the required and optional arity, and not on keyword args. But there it is.

In any case, I look forward to using `lambda*` more in the future, without speed trepidations. Just say no to rest arguments masquerading as optionals!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

### 5 responses

1. asd says:[8 November 2009 12:49 PM](#d2eafe1a26c2b588eb01459d66cf644deece7dcb)

   Holy crap, and I thought Perl code was unreadable. You are setting new standards.

2. Sam TH says:[8 November 2009 2:53 PM](#edf6a06cf05eea5f7521a00126d769fae7c153a5)

   You should really look at this paper, in this year's Scheme workshop:

   Keyword and Optional Arguments in PLT Scheme

   Matthew Flatt and Eli Barzilay

   http://www.ccs.neu.edu/scheme/pubs/scheme2009-fb.pdf

3. Zeeshan Ali (Khattak) says:[8 November 2009 6:42 PM](#edb15a3ee31924e3fcfca27adf470b6070ce69ca)

   asd, Comparing scheme to fraking Perl? That is outrageous and only represents your utter ignorance. Its a scientific fact that Scheme is the simplest language to learn for a person with no programming background. If you don't believe me, find 10 random people with no programming background and teach scheme to 5 of them and perl (or any language you know of) to the other half and you'll be surprised.

   If you don't know a language, it will be very unreadable for sure unless it resembles other languages that you know already. :)

4. [wingo](http://wingolog.org/) says:[8 November 2009 8:31 PM](#4800d3797e3ec6e14b23458ab4ebdb76e28583cf)

   @asd: Ahaawe! Otashi lesha onawa nee.

   @sam: Looks like a very interesting paper, thanks for the pointer :-)

5. [Arne Babenhauserheide](http://draketo.de) says:[8 April 2014 1:00 PM](#f615042e53369014929e40a0bcb3c8ce20111d4a)

   @Khattak: May I say Python? Especially for non-programmers, first impression counts. And for Scheme it often is “what about all those parens???”

Comments are closed.
