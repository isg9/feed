---
title: Elisp Running Time Macro
url: https://nullprogram.com/blog/2009/05/28/
published: "2009-05-28T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/05/28/
---

# Elisp Running Time Macro

## [Elisp Running Time Macro](/blog/2009/05/28/)

May 28, 2009

nullprogram.com/blog/2009/05/28/

I wanted an elisp macro that could measure the running time of a block of code. Specifically, I wanted it to work like this,

```
(measure-time
  ...
  body
  ...)

```

And it would return the running time as seconds in floating point. Well, here's a macro that does it!

```cl
;; ID: 6a3f3d99-f0da-329a-c01c-bb6b868f3239
(defmacro measure-time (&rest body)
  "Measure and return the running time of the code block."
  (declare (indent defun))
  (let ((start (make-symbol "start")))
    `(let ((,start (float-time)))
       ,@body
       (- (float-time) ,start))))
```

It's only good for up to around 18 hours, then the time integer overflows. If only Emacs had arbitrary precision numbers. Here it is in action using my [binomial function from
last week](/blog/2009/05/23).

```cl
(measure-time
  (nck 20 10)
  (nck 30 7))
```

Which, just now, returned `3.643713` seconds when executed.

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Elisp%20Running%20Time%20Macro) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Elisp+Running+Time+Macro).

« [Unquoted Let](/blog/2009/05/24/)

» [Elisp Wishlist](/blog/2009/05/29/)
