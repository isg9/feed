---
title: Programs as Elisp Macros
url: https://nullprogram.com/blog/2012/09/21/
published: "2012-09-21T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/09/21/
---

# Programs as Elisp Macros

## [Programs as Elisp Macros](/blog/2012/09/21/)

September 21, 2012

nullprogram.com/blog/2012/09/21/

This evening I came across an interesting idea: [using system programs as functions](http://sunng.info/blog/2012/09/shake-every-program-can-be-a-clojure-function/). The original idea goes to [`sh`](http://amoffat.github.com/sh/index.html), a Python module that exposes system programs as functions. There’s also a Clojure library called [`shake`](https://github.com/sunng87/shake/) to do the same thing in Clojure.

Thanks to symbols, I think the idea maps especially well onto Lisp because arguments don’t need to be provided as strings. Here are some examples,

```
(ls -lh)
(uname -a)
(cat /etc/debian_version)
(git checkout -b foo)

```

It’s easy to achieve the same effect in Elisp,

```
;;; -*- lexical-binding: t; -*-
(require 'cl)

(defun make-shell-macro (program)
  (fset program
        (cons 'macro
              (lambda (&rest args)
                `(with-temp-buffer
                   (funcall #'call-process
                            ,(symbol-name program) nil t nil
                            ,@(mapcar #'prin1-to-string args))
                   (buffer-string))))))

(let ((path (mapcan #'directory-files (parse-colon-path (getenv "PATH")))))
  (dolist (program (remove-if (lambda (f) (member f '("." ".."))) path))
    (let ((symbol (intern program)))
      (unless (fboundp symbol)
        (make-shell-macro symbol)))))

```

Evaluating the above will install macros for all programs in your `PATH`, except where you already have functions or macros defined. I messed up on the latter point while writing this and broke Emacs enough to require a restart. The system program is called synchronously and the output is returned as a string.

However, *because* arguments aren’t evaluated (macros) this has limited usefulness. These function calls are static and can’t be passed variable arguments. In order to do that arguments would need to be evaluated and symbols would need to be quoted. For example,

```
(defun git-checkout (branch)
  (git 'checkout branch))

(defun ls-l (file)
  (ls '-l file))

```

So I think I’d prefer this interface to the one provided by Clojure’s `shake` (and my Elisp code at the top). I have little need to call programs with static arguments.

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Programs%20as%20Elisp%20Macros) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Programs+as+Elisp+Macros).

« [Elisp Recursive Descent Parser (rdp)](/blog/2012/09/20/)

» [Emacs Abnormal Termination](/blog/2012/09/28/)
