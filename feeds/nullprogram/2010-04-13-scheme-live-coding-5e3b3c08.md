---
title: Scheme Live Coding
url: https://nullprogram.com/blog/2010/04/13/
published: "2010-04-13T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/04/13/
---

# Scheme Live Coding

## [Scheme Live Coding](/blog/2010/04/13/)

April 13, 2010

nullprogram.com/blog/2010/04/13/

[Live coding](http://en.wikipedia.org/wiki/Live_coding) (or livecoding) is software development as a performance art. A programmer's screen is viewed by the audience and a program is written and modified so that it produces sound and maybe even visual effects. The audience gets to see the code and its effects live. I'm not sure if "live" refers to the audience, the editing of live code, or maybe both. There are videos all over the web of this in action so if you haven't seen it yet do a quick search and watch one.

It's fairly easy to obtain livecoding software. For example, there's [Fluxus](http://www.pawfal.org/fluxus/), which extends PLT Scheme to support livecoding.

I've never done livecoding myself, but something I've noticed is that Scheme seems to be a popular choice of language for livecoding. I think I know how and why this is. Scheme, being a Lisp dialect, is naturally a [living system](http://steve-yegge.blogspot.com/2007/01/pinocchio-problem.html): it can be modified and extended while it's actively running. Scheme in particular is very well suited for the task thanks to its simplicity and optimized tail recursion.

I'll do a little text-based livecoding example in PLT Scheme to show how it works. This will be easier to do yourself if your text editor can interact directly with a REPL (like Emacs or DrScheme).

Let's define a function that prints a line of text to the screen and recurses, so that it continues printing forever. The recursion is important here and I'll get back to it. To keep things manageable I'm also going to insert a 1 second pause.

```scheme
(define (print-str)
  (display "Hello!\n")
  (sleep 1)
  (print-str))
```

If we call this function with `(print-str)` it will sit there printing "Hello!" over and over. It will also lock up our REPL preventing us from doing anything else. Not very useful. So let's put it in a thread instead!

```scheme
(thread print-str)
```

Now our program is running and we get to keep our REPL. Why do we need to keep our REPL alive? Well, so we can redefine `print-str` on the fly! In my buffer I'll go back and change "Hello!" to "Goodbye!". While I'm doing this the function is still spitting out "Hello!".

```cl
(define (print-str)
  (display "Goodbye!\n")
  (sleep 1)
  (print-str))
```

As soon as I tell my editor to pass this to the REPL the `print-str` function gets redefined and starts printing "Goodbye!" instead. Why did the running function change? Because of recursion. When it called itself, it actually called the new definition.

Since I didn't keep a handle on the thread the easiest way to stop `print-str` from running is to redefine it without recursion.

```cl
(define (print-str)
  (display "Done.\n"))
```

And it's done. If I was really fast about this my output looks something like this.

```
Hello!
Hello!
Hello!
Goodbye!
Goodbye!
Done.

```

That's the fundamental workings of livecoding in Scheme: I changed a program while it was running. To turn the above into the more interesting livecoding you see in the videos all we need are some audio and visual bindings (which is the hard part of it all).

- [lisp](/tags/lisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Scheme%20Live%20Coding) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Scheme+Live+Coding).

« [Emacs cat-safe](/blog/2010/03/31/)

» [The Problem with String Stored Regex](/blog/2010/04/23/)
