---
title: post-harvest moon — wingolog
url: https://wingolog.org/archives/2005/11/13/post-harvest-moon
published: "2005-11-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/11/13/post-harvest-moon
---

# post-harvest moon — wingolog

## [post-harvest moon](/archives/2005/11/13/post-harvest-moon)

13 November 2005 11:42 PM

- [random](/tags/random)
- [python](/tags/python)
- [miyamoto](/tags/miyamoto)

**harvest time on maggie's farm**

Finally got a [Flumotion release](http://flumotion.net/) out last week. Hounding us until the end was a terrible bug manifesting itself as random connection loss between the different processes in the server, and 100% CPU usage that couldn't be traced to anything. None of the profilers I tried (or wrote!) gave any clue as to what was up.

The problems were solved when we switched away from forking out job processes to doing fork+exec, which is a more supported model in Twisted. It could be that we weren't correctly cleaning out all of the file descriptors in the child's main loops implementing the reactor in Twisted, causing processes to wake up all the time. It is difficult to analyze exactly what is the state that needs cleaning up in a program like that, instead of analyzing exactly what state to keep for the new process. Also, exarkun in `#twisted` had an interesting observation, which was that with python's refcounting gc, just about every time you access a variable you modify its refcount, which requires that the child have its own copy of that memory page. Really takes away the copy-on-write advantages of forking processes.

The moral of the story would be that usually you don't want to fork in Python. Changing this to execute separate processes took about 4 hours one afternoon, and took away just about all of the bugs we had been seeing. Thank Jesus!

Also those four hours were krazy 4G1L3. Only thing was it was on my machine, and I have focus-follows-mouse, a different keyboard from [Thomas](http://thomas.apestaart.org/), swapped caps and control, and I use emacs and he uses vi. But somehow we limped along. Agile limping.

**very crucially serious notes**

A [perspicacious analysis](http://qwantz.com/index.pl?comic=192) of [the advantages of being a nice person](http://thomas.apestaart.org/log/index.php?p=324).

Also this is the **crucial phrase of this sentence**. I think this convention is so great I'm going find a **professional typographer** to ask what they think about it. It's about time that some of the [popular exponents](http://www.theonion.com/content/harvey/blog) of this writing style get their **well-deserved recognition**!

Other things I should write about at some point: more notes on using [baz](http://bazaar.canonical.com/) and [arch-pqm](http://web.verbum.org/arch-pqm/), my very pleasant impressions of [SLIME](http://common-lisp.net/project/slime/) and [SBCL](http://sbcl.sourceforge.net/), a recent Aikido seminar with Miyamoto sensei (coming all the way from [Hombu dojo](http://www.aikikai.or.jp/)), an upcoming trip to the states.

## related articles

- [buggin out](/archives/2005/09/08/buggin-out)
- [Inversion of inversion of control](/archives/2005/04/06/100)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)

Comments are closed.
