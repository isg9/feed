---
title: Brainfuck Halting Problem
url: https://nullprogram.com/blog/2009/04/12/
published: "2009-04-12T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/04/12/
---

# Brainfuck Halting Problem

## [Brainfuck Halting Problem](/blog/2009/04/12/)

April 12, 2009

nullprogram.com/blog/2009/04/12/

On my [brainfuck compiler
project](https://github.com/skeeto/bfc), I proposed pre-calculation as an optimization technique. The idea can work, but it has an issue that will always be unsolvable: how do you know that the pre-calculation will halt? This is called [the
halting problem](http://en.wikipedia.org/wiki/Halting_problem) and it has been proven impossible to solve.

The idea was that the compiler would run the brainfuck program up until the first input operation — if there even was one. It would record all output and the final state of the memory. Instead of compiling the code was was run, it would compile code that would print all of the output and set the memory at the final state.

I has mistakenly assumed that it would be possible to detect a non-halting program and avoid doing pre-calculation on it. I described how it would be done and left it at that. Recently, someone kindly sent me an email containing only 5 letters:

```
+[--]

```

This defeated my ill-conceived idea.

Because brainfuck is Turing complete, it is actually impossible to determine whether or not an arbitrary brainfuck loop will halt. A computer can't do it. A human brain (a fancy computer) can't do it either. It cannot be done, at least not in this universe.

So, if implemented, this pre-calculation measure will always be flawed.

- [math](/tags/math/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Brainfuck%20Halting%20Problem) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Brainfuck+Halting+Problem).

« [The Lazy Fibonacci List](/blog/2009/04/10/)

» [URL Shortening](/blog/2009/04/16/)
