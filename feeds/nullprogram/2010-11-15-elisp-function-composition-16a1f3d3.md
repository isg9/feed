---
title: Elisp Function Composition
url: https://nullprogram.com/blog/2010/11/15/
published: "2010-11-15T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/11/15/
---

# Elisp Function Composition

## [Elisp Function Composition](/blog/2010/11/15/)

November 15, 2010

nullprogram.com/blog/2010/11/15/

During my recent Elisp hacking I've run into the situation enough times where I really wanted function composition that I officially implemented it for myself. While there is an [`apply-partially`](/blog/2010/09/29/) function, Elisp does not currently come with a `compose` function. Here's an Elisp definition,

```cl
;; ID: f0c736a9-afec-3e3f-455c-40997023e130
(defun compose (&rest funs)
  "Return function composed of FUNS."
  (lexical-let ((lex-funs funs))
    (lambda (&rest args)
      (reduce 'funcall (butlast lex-funs)
              :from-end t
              :initial-value (apply (car (last lex-funs)) args)))))
```

Here it is in action with three functions.

```cl
(funcall (compose 'prin1-to-string 'random* 'exp) 10)
```

I'll be using this in later posts (and linking back here when I do).

- [elisp](/tags/elisp/)
- [emacs](/tags/emacs/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Elisp%20Function%20Composition) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Elisp+Function+Composition).

« [Sudoku Applet](/blog/2010/10/20/)

» [Pth and ncurses](/blog/2010/12/21/)
