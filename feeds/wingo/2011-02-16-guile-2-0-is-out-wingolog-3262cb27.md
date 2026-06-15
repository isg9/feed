---
title: guile 2.0 is out! — wingolog
url: https://wingolog.org/archives/2011/02/16/guile-is-out
published: "2011-02-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/02/16/guile-is-out
---

# guile 2.0 is out! — wingolog

## [guile 2.0 is out!](/archives/2011/02/16/guile-is-out)

16 February 2011 11:06 AM

- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [scheme](/tags/scheme)

Hear ye, hear ye: [Guile 2.0 is unleashed upon the world!](http://article.gmane.org/gmane.lisp.guile.devel/11654)

[Guile](http://www.gnu.org/software/guile/) is a great implementation of Scheme. It's a joyful programming environment that gives the hacker expressive tools for growing programs.

This release improves Guile tremendously, adding a new compiler and virtual machine, and integrating more powerful hygienic macros in the core of Guile. We managed to pull all of this off with minimal incompatibilities with old code, and maximal awesomeness.

Guile 2.0 is a personal milestone as well, as it is my first stable release as a Guile co-maintainer. I would like to express special thanks to Ludovic Courtès, my co-maintainer, for companionship in the fantastic hack over the last few years. I would also like to express a strange thanks to two people whom I have never met in person, but that, through their code artifacts, really made Guile 2.0 what it is. To Keisuke Nishida, for his initial compiler and VM [ten years ago](http://sources.redhat.com/ml/guile/2000-07/msg00418.html), for teaching me how to write a compiler; and to R. Kent Dybvig, for his [psyntax macro expander](https://www.cs.indiana.edu/chezscheme/syntax-case/old-psyntax.html), for teaching me about macro expanders. Thanks!

Check the [release announcement](http://article.gmane.org/gmane.lisp.guile.devel/11654) for a short summary of changes, or [NEWS](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=NEWS;hb=v2.0.0) for all the delicious details. Start with the [manual](http://www.gnu.org/software/guile/manual/) if you like; it's a great read. Then [download and build](http://www.gnu.org/software/guile/download.html) the thing, and when next you start it up, just think: this could be the beginning of a beautiful hack!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

### 5 responses

1. djcb says:[16 February 2011 11:55 AM](#8dc6510318ecdfadc26eccb4e6d4acb130e8d7a7)

   Awesome! Thanks for the great work, Andy!

   Next step: getting emacs to use it...

2. mvo says:[16 February 2011 12:29 PM](#8b35fbe006a6b1fc43b14209110e212a461608e4)

   Fantastic!

3. Zeeshan Ali (Khattak) says:[16 February 2011 2:00 PM](#0afc65950125a21ca21bfde8cd98987daf36d5a5)

   Finally!

4. [Julian Graham](http://undecidable.net/joolean/) says:[16 February 2011 2:17 PM](#8211440aac3201f9a3fd96ae1ad2789b50bddc41)

   Congratulations, Andy, and thanks for all your hard work!

5. fds says:[16 February 2011 10:52 PM](#e96db9ebe1bb4e5bddb156322ae533469dd4e1d7)

   This is a historic achievement indeed!

   Many thanks to you, Ludovic and everyone else who has contributed to Guile over the years, keep up the good work!

Comments are closed.
