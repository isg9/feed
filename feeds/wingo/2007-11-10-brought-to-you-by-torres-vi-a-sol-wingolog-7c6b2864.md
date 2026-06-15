---
title: brought to you by torres viña sol — wingolog
url: https://wingolog.org/archives/2007/11/10/brought-to-you-by-torres-vina-sol
published: "2007-11-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/11/10/brought-to-you-by-torres-vina-sol
---

# brought to you by torres viña sol — wingolog

## [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)

10 November 2007 11:31 PM

- [scheme](/tags/scheme)
- [gnome](/tags/gnome)
- [guile](/tags/guile)
- [documentation](/tags/documentation)

**guile-gnome**

Released a new version of [guile-gnome](http://www.gnu.org/software/guile-gnome/); release notes [sent to the mailing list](http://article.gmane.org/gmane.lisp.guile.gtk/730).

I used some techniques I [wrote about previously](//wingolog.org/archives/2007/07/18/documenting-language-bindings) to generate [schemey texinfo](http://www.gnu.org/software/guile-gnome/docs/) from upstream's [docbook for C](http://library.gnome.org/devel/references), with a twist: when generating function docs, we load up the wrapset metadata, and use that to determine which functions are actually in the wrapset, what their arguments are, and if they have a generic function associated with them.

Similarly, when documenting GObject classes, we can actually introspect the GObject class from Scheme, loading it up at runtime to query its properties and signals.

The results are pretty good, although sometimes the C conventions show through unnecessarily; we need to go through and provide custom documentation for those procedures using a method similar to that used in [guile-cairo](http://codebrowse.launchpad.net/~andywingo/guile-cairo/trunk/annotate/wingo%40pobox.com-20070924193049-042d4d7demn3ok21?file_id=readme-20070728125845-f8w5ckz1p419lsyk-1).

The generation of docs only occurs manually, because it depends on the location of files on your filesystem, and some libraries not required for the normal build. The upshot is that the tarballs just have texinfo, which has a well-deployed toolchain.

Folks actually visiting the documentation for the first time might wonder why we wrap such an old version of the GNOME platform, 2.16. The reason is that to stabilize the bindings, we needed to focus on a particular GNOME revision. Once the bindings become stable, updating the versions of the wrapped libraries and wrapping the new functions is relatively easy.

One thing that I really wanted to do, and finally have done for this release, is to integrate API regression unit tests into the build. This was done in two parts: [apicheck, a tool to describe Schem APIs and API differences](http://home.gna.org/guile-lib/doc/ref/apicheck/), and then integrating apicheck into the various wrapset modules, a process started in the last release. For example, part of the API description for `(gnome gtk)`, as produced by `apicheck-generate`, looks like this:

```
(module-api
  (version 1 0)
  ((gnome gtk)
   (uses-interfaces (gnome gw gdk) (gnome gw gtk))
   (typed-exports
     ( class)
     (create-tag
       generic
       (  . ))
     (gtk-stock-id procedure (arity 1 0 #f))
     (gtk-text-buffer-create-tag
       procedure
       (arity 2 0 #t))
     ...))
  ((gnome gw gdk)
   (uses-interfaces (gnome gw generics))
   (typed-exports
     (gdk-atom-name procedure (arity 0 0 #t))
     (gdk-beep procedure (arity 0 0 #t))
     (gdk-cairo-create procedure (arity 1 0 #f))
     (gdk-cairo-rectangle procedure (arity 2 0 #f))
     ...))
  ...)

```

Pretty simple in the end -- set differences, with specialized calculation on whether two [arities](http://en.wikipedia.org/wiki/Arity) or sets of methods on generics are compatible. Anyway I thought this was neat and the wine told me to write about it.

The message that the wine would like me to bring you is this: the next time you start a project, think about how you've been wanting to learn Scheme, and remember that there's a reasonably well-documented, reasonably stable binding for Guile Scheme, and that maybe you should give it a try. Let me know how it goes!

## related articles

- [documenting language bindings](/archives/2007/07/18/documenting-language-bindings)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [go put it in a washing machine](/archives/2007/09/10/go-put-it-in-a-washing-machine)
- [happy birthday gnome!](/archives/2007/08/14/happy-birthday-gnome)
- [foreign interfaces](/archives/2007/05/24/foreign-interfaces)
- [\-\\*\- text -\*-](/archives/2007/05/22/text)

Comments are closed.
