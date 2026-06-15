---
title: case-lambda in guile — wingolog
url: https://wingolog.org/archives/2009/11/07/case-lambda-in-guile
published: "2009-11-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/11/07/case-lambda-in-guile
---

# case-lambda in guile — wingolog

## [case-lambda in guile](/archives/2009/11/07/case-lambda-in-guile)

7 November 2009 11:52 AM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [case-lambda](/tags/case-lambda)
- [dispatch](/tags/dispatch)

Oh man, does the hack proceed apace. I really haven't had time to write about it all, but stories don't tell themselves, so it's back behind the megaphone for me.

[Guile](http://www.gnu.org/software/guile/guile.html) is doing well, with the monthly release train still on the roll. Check the latest [news entries](http://www.gnu.org/software/guile/news.html) for the particulars of the past; but here I'd like to write about a couple aspects of the present.

First, `case-lambda`. The dilly here is that sometimes you want a procedure that can take N or M arguments. For example, Scheme's `write` can be invoked as:

```
(write "Hi." (current-output-port))
=| "Hi."

```

( `=|` means "prints", in the same way that `=>` means "yields".)

But actually you can omit the second argument, because it defaults to the current output port anyway, and just do:

```
(write "Hi.")
=| "Hi."

```

Well hello. So the question: how can one procedure take two different numbers of arguments -- how can it have two different [arities](http://en.wikipedia.org/wiki/Arity)?

The standard answer in Scheme is the "rest argument", as in "this procedure has two arguments, and put the rest in the third." The syntax for it is not very elegant, because it introduces improper lists into the code:

```
(define (foo a b . c)
  (format #t "~a ~a ~a\n" a b c))
(foo 1 2 3 4)
=| 1 2 (3 4)

```

You see that 1 and 2 are apart, but that 3 and 4 have been consed into a list. Rest args are great when your procedure really does take any number of arguments, but if the true situation is that your procedure simply takes 1 or 2 arguments, you end up with code like this:

```
(define my-write
  (lambda (obj . rest)
    (let ((port (if (pair? rest)
                    (car rest)
                    (current-output-port))))
      (write obj port))))

```

It's ugly, and it's not expressive. What's more, there's a bug in the code above -- that you can give it 3 arguments and it does not complain. And even more than that, it actually has to allocate memory to store the rest argument, on every function call. (Whole-program analysis can recover this, but that is an entirely different kettle of fish.)

The solution to this is `case-lambda`, which allows you to have one procedure with many different arities.

```
(define my-write
  (case-lambda
    ((obj port) (write obj port))
    ((obj)      (my-write obj (current-output-port)))))

```

**implementation**

You can implement case-lambda in terms of rest arguments, with macros. Guile did so for many years. But you don't get the efficiency benefits that way, and all of your tools still assume functions only have one arity.

Probably the first time you make a VM, you encode the arity of a procedure into the procedure itself, in some kind of header. Then the opcodes that do calls or tail-calls or what-have-you check the procedure header against the number of arguments, to make sure that everything is right before transferring control to the new procedure.

Well with case-lambda that's not a good idea. Actually if you think a bit, there are all kinds of things that procedures might want to do with their arguments -- optional and keyword arguments, for example. (I'll discuss those shortly.) Or when you are implementing Elisp, and you have a rest argument, you should make a nil-terminated list instead of a null-terminated list. Et cetera. Many variations, and yet the base case should be fast.

The answer is to make calling a procedure very simple -- just a jump to the new location. Then let the procedure that's being called handle its arguments. If it's a simple procedure, then it's a simple check, or if it's a case-lambda, then you have some dispatch. Indeed in Guile's VM now there are opcodes to branch based on the number of arguments.

So much for the VM; what about the compiler and the toolchain? For the compiler it's got its ups and downs. Instead of a `that just has its arguments and body, it now has no arguments, and a` as its body. Each lambda-case has an "alternate", the next one in the series. More complicated.

Then you have the debugging information about the arities. The deal here is that there are parts of a procedure that have arities, probably contiguous parts, and there are parts that have no arity at all. For example, program counter 0 in most procedures has no arity -- no bindings have been made from the arguments to local variables -- because the number of arguments hasn't been checked yet. And if that check fails, you'll want to show those arguments on your stacktrace. Complication there too.

And the introspection procedures, like `procedure-arguments` and such, all need to be updated. On the plus side, and this is a big plus, now there is much more debugging information available. Argument names for the different case-lambda clauses, and whether they are required or rest arguments -- and also optional and keyword arguments. This is nice. So for example `my-write` prints like this:

```
#

```

So yeah, Guile does efficient multiple-arity dispatch now, and has the toolchain to back it up.

Next up, efficient optional and keyword arguments. Tata for now!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

Comments are closed.
