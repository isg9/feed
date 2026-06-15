---
title: Wisp Screencasts
url: https://nullprogram.com/blog/2010/02/04/
published: "2010-02-04T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/02/04/
---

# Wisp Screencasts

## [Wisp Screencasts](/blog/2010/02/04/)

February 04, 2010

nullprogram.com/blog/2010/02/04/

I've been chugging away on Wisp, [announced
in my last post](/blog/2010/01/24/), every day since I started it a few weeks ago, and it's becoming a pretty solid system. There's now an exception system, reference counting for dealing with garbage, and a reentrant parser. It's no replacement for any other lisps, but I've found it to be very fun to work on.

```
git clone git://github.com/skeeto/wisp.git

```

I wanted to show off some of the new features of Wisp, and since I was inspired by [Full
Disclojure](http://vimeo.com/channels/fulldisclojure), since it's so damn slick, I decided to make some screencasts of Wisp in action. All of the screencast software for GNU/Linux is pretty poor, but after a few hours of head-banging I managed to hobble something together for you. Enjoy!

Since your browser doesn't seem to support the video tag, here's a link to the video: [wisp-memoize.ogv](/vid/wisp/wisp-memoize.ogv).

That video demonstrated the memoization function. It can be pulled in from the `memoize` library. You give it a symbol, which should have a function definition stored in it, and it will installed a wrapper around it. In the video I used the Fibonacci function from the `examples` library.

```
(require 'examples)
(fib 30) ; Slooooow ...
(memoize 'fib)
(fib 100) ; Fast!

```

Since your browser doesn't seem to support the video tag, here's a link to the video: [wisp-detach.ogv](/vid/wisp/wisp-detach.ogv).

This demonstrated the "detachment" feature of Wisp, which is similar to "futures" in Clojure. It forks off a new process, which executes the given function. The `send` function can be used in the detached process to send any lisp objects back to the parents, which can receive them with the `receive` function. The `send` function can be called any number of times to continually send data back. The `receive` function will block if there is no lisp object to receive yet.

```
(require 'examples)
(setq d (detach (lambda () (send (fib)))))
(receive d) ; Gets value from child process

```

Since your browser doesn't seem to support the video tag, here's a link to the video: [wisp-point-free.ogv](/vid/wisp/wisp-point-free.ogv).

This video shows off the point-free functions that have been defined: function composition and partial application (I accidentally say "partial evaluation" in the video). These are actually just simple macros that any lisp could do.

- [video](/tags/video/)
- [lisp](/tags/lisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Wisp%20Screencasts) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Wisp+Screencasts).

« [Wisp Lisp](/blog/2010/01/24/)

» [Common Lisp Quick Reference](/blog/2010/02/06/)
