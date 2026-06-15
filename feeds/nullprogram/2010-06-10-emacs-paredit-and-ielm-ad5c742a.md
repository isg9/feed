---
title: Emacs ParEdit and IELM
url: https://nullprogram.com/blog/2010/06/10/
published: "2010-06-10T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/06/10/
---

# Emacs ParEdit and IELM

## [Emacs ParEdit and IELM](/blog/2010/06/10/)

June 10, 2010

nullprogram.com/blog/2010/06/10/

[ParEdit](http://www.emacswiki.org/emacs/ParEdit) is a powerful extension to Emacs that I've just begun using recently. It's a minor mode that forces all parenthesis, square brackets, and quotes to be balanced at all times. While it's useful for any programming language it's especially suited for Lisps, because it's designed for manipulating nested parenthesis — i.e. s-expressions. It's not currently part of Emacs so you have to drop the script in your `load-path` somewhere.

I've frequently thought that a Lisp-based shell would be an interesting and powerful tool, much like a normal Lisp REPL. Programs would be treated like Lisp functions. For example,

```
wellons@luna:~$ (ls -l .emacs)
-rw------- 1 wellons wellons 4859 2010-06-10 23:20 .emacs
wellons@luna:~$

```

But typing all those parenthesis all the time would be quite the nuisance. I know this from experience typing at Lisp REPLs. I imagined something that works exactly like ParEdit would be needed to make all that work go away. To save even more time each prompt would begin with a nested pair, with the cursor placed between them. Then typing a quick command is no different than a normal shell.

```
wellons@luna:~$ ()

```

Well, in Emacs we have both ParEdit and REPLs, so we can compose these features together with just a little advice. Here's how to do it with the Interactive Emacs-Lisp Mode (IELM) REPL. First tell IELM to use ParEdit,

```cl
(add-hook 'ielm-mode-hook (lambda () (paredit-mode 1)))
```

The function in IELM that spits out the next prompt is `ielm-eval-input`, so we give it the advice to call the ParEdit function afterwards to insert a parenthesis pair.

```cl
(defadvice ielm-eval-input (after ielm-paredit activate)
  "Begin each IELM prompt with a ParEdit parenthesis pair."
  (paredit-open-round))
```

And that's it! Note that the first IELM prompt is not placed by this function so it won't appear until the second prompt.

```
*** Welcome to IELM ***  Type (describe-mode) for help.
ELISP>
ELISP> ()

```

If you want to enter a single atom and don't need parenthesis, just hit backspace once. This is much less common so it gets the extra keystroke.

This can be done for `inferior-lisp` and [SLIME](/blog/2010/01/15/) to enhance those REPLs as well. You just have to figure out which defun to advise.

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs%20ParEdit%20and%20IELM) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs+ParEdit+and+IELM).

« [Elisp Printed Hash Tables](/blog/2010/06/07/)

» [Emacs Byte Compilation](/blog/2010/07/01/)
