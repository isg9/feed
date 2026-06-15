---
title: breaking silence, innocuously — wingolog
url: https://wingolog.org/archives/2008/09/18/breaking-silence-innocuously
published: "2008-09-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/09/18/breaking-silence-innocuously
---

# breaking silence, innocuously — wingolog

## [breaking silence, innocuously](/archives/2008/09/18/breaking-silence-innocuously)

18 September 2008 10:17 PM

- [guile](/tags/guile)
- [compilation](/tags/compilation)
- [scheme](/tags/scheme)

Greeting, faerie webfolk.

I have a number of things in yon backlog, but as they build up, they stop up, so I figured with a bottle of wine behind me I could loose the gates on a topic: continuations.

Now, friends of mine: you may not be interested in this topic. If this is the case, treat this as the gristle: it is perhaps not tasty, but it comes with the meat.

But if the topic of call/cc and continuations interests you, you may treat this as the fat in [panceta](http://en.wikipedia.org/wiki/Pancetta) (I do not agree with their spelling): delicious.

If you bristle at the double-dot: that's ok too.

My point when I started all of this was, Scheme weenies extol the benefits of `call-with-current-continuation`. (Not until the most recent standard, the controversial [r6rs](http://www.r6rs.org/), did the establishment deign to accept the popular spelling, `call/cc`. Such is the Scheme culture.) Weenies of other languages (Ruby, although Google denies me the references: searching while tipsy, the cop said) also relish in their esoteric delight.

But with all of the literature in the Scheme community about [continuations and compilation](http://library.readscheme.org/page6.html), with all the talk about continuation-based web platforms, it's still basically intractable.

The reasons are two, basically. One, you cannot represent all of the continuation of an operation; there is always a level beyond which you cannot represent. This is most basically evident in the state of the connection of the continuation-aware system to the outside world, for example as indicated in the file descriptor table in the kernel.

The second reason is that you almost always grab more information than you want, making call/cc a quite heavy-weight operation. The most brute-force method simply copies the entire C stack from the point of the call/cc on down to some root address, and reinstates on reinvocation -- yet you almost never touch the bottom stack frames in such a continuation. I think stack-copying is a perfectly valid implementation strategy for continuations, in many contexts, but not of the entire C stack down to arbitrary depth.

Now, *delimited* continuations and prompts: *there* is where the actual useful possibilities are. As always, [Oleg](http://okmij.org/ftp/Computation/Continuations.html#context-OS) has lots to say on that, including a number of very interesting analogies to processes and operating systems.

I suppose it was Felleisen that first came up with the analogy, and I'm sure he was thinking of it in operating-system contexts -- the REPL is after all a boundary between an operating system and user-space. I'd link to his paper if the ACM weren't a pack of scrooge mcducks. ( *The theory and practice of first-class prompts*, if that's your thing.)

This post brought to you by Miguel Torres, S.A and the need to compile [r4rs.scm](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=ice-9/r4rs.scm;h=de2aeb2def52a53f0bcc0fcc45242b5ee55d5e39;hb=refs/heads/vm), about which more later.

## related articles

- [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x)))](/archives/2008/11/14/lambda-x-compile-x-x-lambda-x-compile-x-x)
- [guile bar mitzvah](/archives/2008/11/02/guile-bar-mitzvah)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

Comments are closed.
