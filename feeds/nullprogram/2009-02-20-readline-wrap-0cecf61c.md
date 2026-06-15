---
title: Readline Wrap
url: https://nullprogram.com/blog/2009/02/20/
published: "2009-02-20T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/02/20/
---

# Readline Wrap

## [Readline Wrap](/blog/2009/02/20/)

February 20, 2009

nullprogram.com/blog/2009/02/20/

I came across a very interesting tool the other day called [rlwrap](http://utopia.knoware.nl/~hlub/rlwrap/). It wraps the readline library over just about any interactive text input program. The readline library provides basic editing and history. It's handy for those programs that don't provide their own line editing facilities.

It tries to be as transparent as possible, detecting yes/no prompts and passwords, so it should still be reasonable under those conditions.

If you can't think of anything to try it with, try it with `cat`. Instant line editor!

```
rlwrap cat > some-file.txt

```

Or with Festival.

```
rlwrap festival --tts

```

It will also turn incorrectly compiled shells on your system into something usable. On my system (Debian GNU/Linux), `csh` isn't usable without `rlwrap`.

```
rlwrap csh

```

- [trick](/tags/trick/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Readline%20Wrap) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Readline+Wrap).

« [Distributed Issue Tracking](/blog/2009/02/14/)

» [GNU Screen](/blog/2009/03/05/)
