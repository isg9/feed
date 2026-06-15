---
title: Jump to Java Documentation from Emacs
url: https://nullprogram.com/blog/2010/10/14/
published: "2010-10-14T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/10/14/
---

# Jump to Java Documentation from Emacs

## [Jump to Java Documentation from Emacs](/blog/2010/10/14/)

October 14, 2010

nullprogram.com/blog/2010/10/14/

Update January 2013: this package has been refined and formally renamed to [javadoc-lookup](https://github.com/skeeto/javadoc-lookup). The user interface is essentially the same — under different function names — but with some extra goodies. It's available for install from MELPA.

I keep running to either a search engine or, when offline, manually browsing to Java API documentation when I need to look something up. When I'm using Emacs this is stupid, so I fixed it. I put together a `java-docs` package that let's me quickly jump to documentation from within Emacs.

Repository: `git clone git://github.com/skeeto/javadoc-lookup.git`

Unfortunately it launches it in a web browser right now because there doesn't seem to be a reasonable way to render the documentation *inside* Emacs itself. So you'll [need to
have `browse-url` set up properly](http://www.emacswiki.org/emacs/BrowseUrl) in your configuration.

I strongly recommend you use this with [Ido](http://www.emacswiki.org/emacs/InteractivelyDoThings), which comes with Emacs. If you do, you'll want to load it *after* you enable `ido-mode`, which will enable the Ido minibuffer completion in `java-docs`.

So, after you `require` `java-docs`, you give it a list of places to look for documentation.

```cl
(require 'java-docs)
(java-docs "/usr/share/doc/openjdk-6-jdk/api" "~/src/project/doc")
```

It will scan these locations and build up an index of classes. [If you're using a recent enough
version of Emacs](/blog/2010/06/07/) it will cache that index for faster loading in the future, since on *certain* systems it can needlessly take a bit of time.

After that you can jump to documentation with `C-h j` ( `java-docs-lookup`). It will ask you what you want to look up and offer completion with your preferred completion function.

[![](/img/emacs/java-import-thumb.png)](/img/emacs/java-import.png)

If you don't want to open it up in an external browser, you can set Emacs to run a text-based browser inside itself.

```cl
(setq browse-url-browser-function 'browse-url-text-emacs)
```

- [java](/tags/java/)
- [emacs](/tags/emacs/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Jump%20to%20Java%20Documentation%20from%20Emacs) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Jump+to+Java+Documentation+from+Emacs).

« [Emacs Set Window to 80 Columns](/blog/2010/10/06/)

» [Introducing Java Mode Plus](/blog/2010/10/15/)
