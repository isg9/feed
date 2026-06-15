---
title: Programmatically Setting Lisp Docstrings
url: https://nullprogram.com/blog/2012/08/02/
published: "2012-08-02T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/08/02/
---

# Programmatically Setting Lisp Docstrings

## [Programmatically Setting Lisp Docstrings](/blog/2012/08/02/)

August 02, 2012

nullprogram.com/blog/2012/08/02/

I just updated my [Elisp memoization function](/blog/2010/07/26/) so that it’s no longer a dirty hack. To work around the lack of closures, due to the lack of lexical scope in Elisp, the original version used uninterned symbols to store the look-up table. The new version in the post uses `lexical-let`, which does the same thing internally to fake a closure. The new version in [my dotfiles](/blog/2011/10/19/) repository uses the brand new [Emacs 24 lexical scoping](http://www.gnu.org/software/emacs/NEWS.24.1).

It was “dirty” because it built a lambda function out of a list at run time, taking advantage of the way Elisp currently handles functions. The reason for this was that I wanted to inject the original documentation string into the new function which can’t normally be done when `lambda` is used the correct way. When I updated the function I fixed this as well. It uses a trick provided by Elisp, which is different than the Common Lisp way that I assumed.

Both Elisp and Common Lisp have a `documentation` function for programmatically accessing symbol documentation. The Elisp version only provides *function* documentation, so it only accepts one argument.

```
(defun foo ()
  "Foo."
  nil)

(documentation 'foo)
=> "Foo."

```

The Common Lisp version must be told what type of documentation to return, such as `function` or `variable` ( `defvar`, `defconst`).

```
(documentation 'foo 'function)
=> "Foo."

```

As it might be expected, this is `setf`-able! It’s possible to update or modify documentation strings without needing to redefine the function.

```
(setf (documentation 'foo 'function) "New doc string.")

```

Unfortunately it’s not `setf`-able in Elisp. Instead you can set the `function-documentation` *property* of the symbol. The `documentation` function will prefer this over the string stored in the function itself.

```
(put 'foo 'function-documentation "Foo updated.")

(documentation 'foo)
=> "Foo updated."

```

The downside is that this is a second place to put docstrings, leading to surprising behavior for developers unaware of this hack.

```
(put 'foo 'function-documentation "Old docstring.")

(defun foo ()
  "New docstring."
  nil)

(documentation 'foo)
=> "Old docstring."

```

This can be fixed by setting the symbol property for `function-documentation` to `nil`.

```
(put 'foo 'function-documentation nil)

```

I prefer the Common Lisp method.

- [elisp](/tags/elisp/)
- [emacs](/tags/emacs/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Programmatically%20Setting%20Lisp%20Docstrings) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Programmatically+Setting+Lisp+Docstrings).

« [Viewing Java Class Files in Emacs](/blog/2012/08/01/)

» [The Physical Analog for Encryption is the Hyperdrive](/blog/2012/08/06/)
