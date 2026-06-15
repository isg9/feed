---
title: high on sodium vapor — wingolog
url: https://wingolog.org/archives/2006/11/17/high-on-sodium-vapor
published: "2006-11-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/11/17/high-on-sodium-vapor
---

# high on sodium vapor — wingolog

## [high on sodium vapor](/archives/2006/11/17/high-on-sodium-vapor)

17 November 2006 8:43 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [texinfo](/tags/texinfo)
- [literate programmin](/tags/literate%20programmin)
- [documentation](/tags/documentation)
- [info](/tags/info)

Hello! I would like to show off a hack. It is this:

[![](//wingolog.org/pub/guile-lib-emacs.png)](http://home.gna.org/guile-lib/doc/ref/sxml.simple/)

This is me editing a piece of [guile-lib](http://home.gna.org/guile-lib/), an anemic collection of modules written in [guile](http://www.gnu.org/software/guile/) scheme. The docstring is highlighted in pink due to a crazy `.emacs` that I inherited from my friend Leif.

Note that the reference to *sxml* is written in [texinfo](http://www.gnu.org/software/texinfo/), what to me is a beautiful and appropriate language for documenting software.

[![](//wingolog.org/pub/guile-lib-texinfo-help.png)](http://home.gna.org/guile-lib/doc/ref/texinfo.reflection/)

This is me at the `guile>` prompt, asking for help on the procedure `sxml->string`. Note that `help` has parsed the argument list and put it there for me, without me having to document it. Also note the the `@var{sxml}` is rendered as `SXML`, which is normal for a [metasyntactic variable](http://www.gnu.org/software/texinfo/manual/texinfo/html_node/var.html).

[![](//wingolog.org/pub/guile-lib-stexi.png)](http://home.gna.org/guile-lib/doc/ref/texinfo/)

That's possible because when I ask for help on an object, guile parses the docstring using [`(texinfo reflection)`](http://home.gna.org/guile-lib/doc/ref/texinfo.reflection/) into an [SXML](http://ssax.sourceforge.net/) dialect.

[![](//wingolog.org/pub/guile-lib-texinfo-html.png)](http://home.gna.org/guile-lib/doc/ref/texinfo.html/)

From the SXML format it's possible to do lots of things, like transforming it into HTML, and then serializing to XML. So we can have nice-looking, clean docs.

[![](//wingolog.org/pub/guile-lib-texinfo-pdf.png)](http://home.gna.org/guile-lib/doc/guile-library.pdf)

Using the parsed texinfo, we can programmatically construct documentation for an entire set of modules, using both the written knowledge in the docs and the "live" knowledge that guile has of the instantiated object. The above image is from a high-quality PDF rendered by TeX, after guile-lib serialized the parsed documentation back to normal texinfo.

![](//wingolog.org/pub/guile-lib-info.png)

The texinfo file can also be processed by `makeinfo`, which is a useful if idiosyncratic system for document browsing. The nice thing about info is its index: press `i sxml-> TAB` and you can see all functions that operate on sxml data.

Neat eh? The only problem with having nice documentation output in many formats is that it doesn't hide bad text. We still have a ways to go in that department.

## related articles

- [documenting language bindings](/archives/2007/07/18/documenting-language-bindings)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

### 4 responses

1. Chris Parker says:[17 November 2006 9:28 PM](#225)

   Wow. I have been playing with Guile a bit, and the guile-gnome bindings are great. Why isn't more software written in it?

2. jay says:[18 November 2006 0:03 AM](#226)

   I'd love to use guile (or scheme/lisp in general) for doing gui stuff for gnome, but to my experience there are not good bindings available :( guile-gnome might be good, but there hasn't been any releases, and the website wasn't updated with news for ages (until now recently, with a post about moving to bzr).

   Hmm there appears to be packages in ubuntu for guile-gnome, but these depend on guile 1.6. Isn't this a very old release? Someone should make sure guile 1.8 is included in the next release of ubuntu, and also make an release of guile-gnome!

3. [Ricky Youngblood](http://www.macgregor26.com/) says:[18 November 2006 5:50 AM](#227)

   When was the last time you got laid?

4. [wingo](http://wingolog.org/) says:[18 November 2006 2:32 PM](#228)

   RICKY YOUNGBLOOD YOU ARE A NEGATIVE INDIVIDUAL

Comments are closed.
