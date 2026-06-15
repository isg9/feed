---
title: A Fractran Short Story
url: https://nullprogram.com/blog/2010/03/09/
published: "2010-03-09T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/03/09/
---

# A Fractran Short Story

## [A Fractran Short Story](/blog/2010/03/09/)

March 09, 2010

nullprogram.com/blog/2010/03/09/

[Fractran](http://en.wikipedia.org/wiki/Fractran) is a Turing-complete esoteric programming language. A Fractran program is just an ordered list of positive, irreducible fractions. The program's output for an input *n* is the output of the program run on *n* multiplied by the first fraction in the list that results in an integer. If no such multiplication results in an integer, the output is the input *n*. Variables are encoded in the exponents of the prime factorization of the input and output.

Some time ago I thought up an idea for a short story involving Fractran. A mathematician accidentally creates a Fractran program that can trivially factor large composites. Think something like O(log *n*). It's just the right magical string of, say, 31 fractions.

The story would be a first-person narrative of the mathematician's thoughts during a short time after the discovery, considering many of the consequences of the program. For example, it would render much of cryptography, which plays an essential role in the modern world, useless. He would also wonder if mankind should deserve such a discovery, considering how accidental it was.

This whole idea vanished once I realized that this Fractran program is actually completely trivial. It even runs in O(1) time. It's so trivial as to be worthless. Remember that Fractran stores its data in the number's prime factorization? The Fractran program that can factor any number in constant time is the identity function. To decode the output, which matches the input, all you need to do is factor it!

Interestingly, it doesn't seem to actually be possible to implement the identity function in Fractran ( *But somehow it's* *Turing-complete? Hmmm... more investigation needed.*), unless you can define your program in terms of its input. For example, the program `1/(n+1)` is the identity function for input *n*.

- [story](/tags/story/)
- [compsci](/tags/compsci/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20A%20Fractran%20Short%20Story) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=A+Fractran+Short+Story).

« [Common Lisp Quick Reference](/blog/2010/02/06/)

» [Emacs cat-safe](/blog/2010/03/31/)
