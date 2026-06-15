---
title: Emacs cat-safe
url: https://nullprogram.com/blog/2010/03/31/
published: "2010-03-31T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/03/31/
---

# Emacs cat-safe

## [Emacs cat-safe](/blog/2010/03/31/)

March 31, 2010

nullprogram.com/blog/2010/03/31/

![](/img/emacs/cat-paw.jpg)

I was inspired by an item in [Luke's](http://www.terminally-incoherent.com/blog/) [Tumblr blog](http://random.terminally-incoherent.com/post/484785491) last night. It was a screenshot of a program called [PawSense](http://www.bitboost.com/pawsense/), which monitors a computer's keyboard for cat activity. (I don't know if it's any good, but it's funny.) As anyone with cats knows, it's not unusual to leave a computer only to come back later to see garbage typed in by a wandering cat. I wrote a version for Emacs today.

```
git clone git://github.com/skeeto/cat-safe.git

```

Put it ( `cat-safe.el`) somewhere in your `load-path` (like `~/.emacs.d/`) and put this line in your `.emacs` file,

```cl
(require 'cat-safe)
```

![Emacs switches focus to a new buffer to stop cat damage.](/img/emacs/cat-safe-thumb.png)

This only monitors Emacs itself; it should help protect your buffers but not your web browser. When cat interference is detected Emacs switches focus to a junk buffer and lets the cat make a mess there instead. In case your cat [happens
to type out some Shakespeare](http://en.wikipedia.org/wiki/Infinite_monkey_theorem) you will be able to read it in the junk buffer. Just kill the junk buffer to return to work.

It could still use some improvement. Right now it looks for a single key being help down, excepting keys humans tend to hold down like backspace, delete, and space. If you play around with it you'll notice if you press several keys at once Emacs will sometimes create a pattern with them. I need to figure out a good way to detect this.

I'm going to run it at home for awhile to make sure it remains transparent, but still does its job. It will probably incur a performance penalty on frequently repeated keyboard macros.

- [emacs](/tags/emacs/)
- [lisp](/tags/lisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs%20cat-safe) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs+cat-safe).

« [A Fractran Short Story](/blog/2010/03/09/)

» [Scheme Live Coding](/blog/2010/04/13/)
