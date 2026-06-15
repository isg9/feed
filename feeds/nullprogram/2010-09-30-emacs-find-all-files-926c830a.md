---
title: Emacs Find All Files
url: https://nullprogram.com/blog/2010/09/30/
published: "2010-09-30T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/09/30/
---

# Emacs Find All Files

## [Emacs Find All Files](/blog/2010/09/30/)

September 30, 2010

nullprogram.com/blog/2010/09/30/

Here's another bit of code I started using recently. I often find myself wanting to open — or reopen after `kill-matching-buffers` — all the files under a specific point in the file system. I'm using it at work now to open up all the source files in a deep Java source tree on small-ish project. Once it's all open I can switch to any file quickly with [ido's fuzzy matching](http://www.emacswiki.org/emacs/InteractivelyDoThings), flattening out the directory structure a bit. (And the ridiculous "security" software at work imposes a 3-second I/O block when opening files, so I get to pay this all up front at once rather than having it later [break my
flow](http://c2.com/cgi/wiki?MentalStateCalledFlow).)

This just recursively travels down the sub-directories opening a buffer for everything it comes across. It ignores dot-files, like the ones your source control might litter.

```cl
;; ID: 72dc0a9e-c41c-31f8-c8f5-d9db8482de1e
(defun find-all-files (dir)
  "Open all files and sub-directories below the given directory."
  (interactive "DBase directory: ")
  (let* ((list (directory-files dir t "^[^.]"))
         (files (remove-if 'file-directory-p list))
         (dirs (remove-if-not 'file-directory-p list)))
    (dolist (file files)
      (find-file-noselect file))
    (dolist (dir dirs)
      (find-file-noselect dir)
      (find-all-files dir))))
```

One caveat: if you have a symbolic link that creates a file system loop, this will probably get hung on it.

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs%20Find%20All%20Files) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs+Find+All+Files).

« [Elisp Higher-order Conversion to Interactive](/blog/2010/09/29/)

» [Sample Java Project](/blog/2010/10/04/)
