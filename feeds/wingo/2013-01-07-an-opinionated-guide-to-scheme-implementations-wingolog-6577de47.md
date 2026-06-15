---
title: an opinionated guide to scheme implementations — wingolog
url: https://wingolog.org/archives/2013/01/07/an-opinionated-guide-to-scheme-implementations
published: "2013-01-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/01/07/an-opinionated-guide-to-scheme-implementations
---

# an opinionated guide to scheme implementations — wingolog

## [an opinionated guide to scheme implementations](/archives/2013/01/07/an-opinionated-guide-to-scheme-implementations)

7 January 2013 1:43 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [chicken](/tags/chicken)
- [racket](/tags/racket)
- [chibi](/tags/chibi)
- [gambit](/tags/gambit)

Hark, the beloved programing language! But hark, also: for Scheme is a language of many implementations. If Scheme were a countryside, it would have its cosmopolitan cities, its hipster dives, its blue-collar factory towns, quaint villages, western movie sets full of façades and no buildings, shacks in the woods, and single-occupancy rent-by-the-hour slums. It's a confusing and delightful place, but you need a guide.

And so it is that there is a steady rivulet of folks coming into the `#scheme` IRC channel asking what implementation to use. Mostly the regulars are tired of the question, but when they do answer it's in very guarded terms -- because the topic is a bit touchy, and a bit tribal. There's a subtle tension between partisans of the different implementations, but people are polite enough to avoid controversial, categorical statements. Unfortunately the end result of this is usually silence in the channel.

Well I'm tired of it! All that silence is boring, and so I thought I'd give you my opinionated guide to Scheme implementations. This is necessarily subjective and colored, as I co-maintain the Guile implementation and do all of my hacking in Guile. But hey, to each folk their stroke, right? What I say might still be valuable to you.

So without further introduction, here are seven recommendations for seven use cases.

**The embedded Scheme**

So you have a big project written in C or C++ and you want to embed a Scheme interpreter, to allow you to extend this project from the inside in a higher-level, dynamic language. This is the sort of thing where [Lua](http://www.lua.org/) really shines. So what Scheme to use?

Use [Chibi Scheme](http://code.google.com/p/chibi-scheme/)! It's small and from all accounts works great for static linking. Include a copy of it in your project, and save yourself the headache of adding an external dependency to something that changes.

Embedding Scheme is a great option if you need to ship an application on Windows, iOS, or Android. The other possibility is shipping a native binary created by a Scheme that compiles natively, so...

**The Scheme that can make a binary executable**

Sometimes what you want to do is just ship an executable, and not rely on the user to install a runtime for a particular Scheme system. Here you should use [Chicken](http://www.call-cc.org/) or [Gambit](http://gambitscheme.org/wiki/index.php/Main_Page). Both of these Schemes have a compiler that produces C, which it then passes on to your system's C compiler to generate an executable.

Another possibility is to use [Racket](http://racket-lang.org/), and use its [`raco exe` and `raco dist`](http://docs.racket-lang.org/raco/exe-dist.html) commands to package together a program along with a dynamically-linked runtime into a distributable directory.

**The dynamically linked Scheme**

What if you want to extend a project written in C, C++, or the like, but you want to dynamically link to a Scheme provided with the user's system instead of statically linking? Here I recommend [Guile](http://gnu.org/s/guile/). Its C API and ABI are stable, parallel-installable, well-documented, and binary packages are available on all Free Software systems. It's LGPL, so it doesn't impose any licensing requirements on your C program, or on the Scheme that you write.

**The Scheme with an integrated development environment**

What if you really like IDEs? For great justice, download you a [Racket](http://racket-lang.org/)! It has a great debugger, an excellent Scheme editor, on-line help, and it's easy to run on all popular platforms. Incidentally, I would say that Racket is probably the most newbie-friendly Scheme there is, something that stems from its long association with education in US high schools and undergraduate programs. But don't let the pedagogical side of things fool you into thinking it is underpowered: besides its usefulness for language research, Racket includes a package manager with lots of real-world modules, and lots of people swear by it for getting real work done.

**The Scheme for SICP**

Many people come by `#scheme` asking which Scheme they should use for following along with [*Structure and Interpretation of Computer Programs*](http://mitpress.mit.edu/sicp/). SICP was originally written for [MIT Scheme](http://www.gnu.org/software/mit-scheme/), but these days it's easier to use Neil Van Dyke's [SICP mode for Racket](http://www.neilvandyke.org/racket-sicp/). It has nice support for SICP's "picture langauge". So do that!

**The server-side Scheme**

So you write the intertubes, do you? Well here you have a plethora of options. While I write all of my server-side software in [Guile](http://gnu.org/s/guile/), and I think it's a fine implementation for this purpose, let me recommend [Chicken](http://www.call-cc.org/) for your consideration. It has loads of modules ("eggs", they call them) for all kinds of tasks -- AWS integration, great HTTP support, zeromq, excellent support for systems-level programming, and they have a good community of server-side hackers.

**The Scheme for interactive development with Emacs**

Install [Guile](http://gnu.org/s/guile/)! All right, this point is a bit of an advertisement, but it's my blog so that's OK. So the thing you need to do is [install Paredit and Geiser](http://www.gnu.org/software/guile/manual/html_node/Using-Guile-in-Emacs.html). That page I linked to gives you the procedure. From there you can build up your program incrementally, starting with debugging the null program. It's a nice experience.

**You and your Scheme**

These recommendations are starting points. Most of these systems are multi-purpose: you can use Gambit as an interpreter for source code, Racket for server-side programing, or Chicken in Emacs via [SLIME](http://wiki.call-cc.org/eggref/4/slime). And there are a panoply of other fine Schemes: [Chez](http://www.scheme.com/), [Gauche](http://practical-scheme.net/gauche/), [Kawa](http://www.gnu.org/software/kawa/), and the list goes [on and on](http://en.wikipedia.org/wiki/Category:Scheme_implementations) (and even farther beyond that). The important thing in the beginning is to make a good starting choice, *invest some time with your choice*, and then (possibly) change implementations if needed.

[To choose an implementation is to choose a tribe](//wingolog.org/archives/2008/07/10/how-to-choose-between-equivalent-options). Since Scheme is so minimal, you begin to rely on extensions that are only present in your implementation, and so through code you bind yourself to a world of code, people, and practice, loosely bound to the rest of the Scheme world through a fictional first-person-plural. This is OK! Going deep into a relationship with an implementation is the only way to do great work. The looser ties to the rest of the Scheme world in the form of the standards, the [literature](http://library.readscheme.org/), the IRC channel, and the mailing lists provide refreshing conversation among fellow travellers, not marching orders for a phalanx.

So don't fear implementation lock-in -- treat your implementation's facilities as strengths, until you have more experience and know more about how your needs are served by your Scheme.

**Finally...**

Just do it! You wouldn't think that this point needs elaboration, but I see so many people floundering and blathering without coding, so it bears repeating. Some people get this strange nostalgic "Scheme is an ideal that I cannot reach" mental block that is completely unproductive, not to mention lame. If you're thinking about using Scheme, do it! Install your Scheme, run your REPL, and start typing stuff at it. And try to write as much of your program in Scheme as possible. [Prefer extension over embedding](http://twistedmatrix.com/users/glyph/rant/extendit.html). Use an existing implementation rather than writing your own. Get good at using your REPL (or the IDE, in the case of Racket). Read your implementation's manual. Work with other people's code so you get an idea of what kind of code experienced Schemers write. Read the source code of your implementation.

See you in `#scheme`, and happy hacking!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [bart and lisa, hacker edition](/archives/2011/03/19/bart-and-lisa-hacker-edition)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [fibs, lies, and benchmarks](/archives/2019/06/26/fibs-lies-and-benchmarks)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)

### 30 responses

1. [Jean-François Héon](http://thecarefulprogrammer.blogspot.ca/) says:[7 January 2013 3:41 PM](#0b1bfe3309022a60750798fe056b437e4de0a041)

   I really like the "Scheme for X" theme.

   If I may be so bold as to widen the theme to "Lisp for X", I would suggest newLISP as the "Lisp for system administration." It is well-documented and well-maintained.

   http://www.newlisp.org/

2. John says:[7 January 2013 3:46 PM](#84d34e60a42826d47e16e7032bb43a4589c71830)

   Chibi Scheme is good for embedding, but not for static linking, as build scripts can only create chibi shared library (https://groups.google.com/forum/?fromgroups=#!topic/chibi-scheme/HF\_0ogN5xZg).

   Btw. it is nice to see Guile API got stabilized; people still have bad taste for Guile embedding due previously often API changes.

   Also, Guile 2.x is not yet favoured by distros, which is pretty sad; e.g. Fedora still uses 1.8, not sure for Ubuntu...

3. rixed says:[7 January 2013 5:53 PM](#36a23e6d45d4aafd58ca25c37fd028b87e97e176)

   No opinion about bigloo? I'd say it is at least as good as gambit or chicken for binary executable delivery...

4. Simon Jones says:[7 January 2013 5:59 PM](#44a44e044178e8e2087fa873948127bf3951755c)

   What about Stalin?

5. [scott f](https://scott.mn) says:[7 January 2013 6:38 PM](#8233167a9138d34359078be5a8e3c8cce6cd215e)

   Great overview, but IMHO you missed out on a big use case: “Scheme for the browser”. There are several implementations that compile to JS listed here under “Lisp, Scheme”: https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS

   The only one I've played with is Biwa, but Spock, which is by the author of Chicken and uses a similar compilation strategy, looks interesting. There's also Fargo, Moby, nconc, scheme2js and Whalesong. I'm not sure how ready for prime-time any of them are. Would be nice to hear about experiences!

6. kar says:[7 January 2013 6:47 PM](#c8bcf4637abb40dc57d53dbc840c695480e6137e)

   Quote: "There's a subtle tension between partisans of the different implementations, but people are polite enough not to avoid controversial, categorical statements."

   I think you meant to leave out that 'not'.

7. [wingo](http://wingolog.org/) says:[7 January 2013 7:41 PM](#4da4eae05120f60c1891bb4c1afd9261aa6e7044)

   kar: fixed, thanks!

8. [John Cowan](http://www.ccil.org/~cowan) says:[7 January 2013 8:21 PM](#0a8fd8166c21c256d9c2ce778c63d14072158ad8)

   NewLisp is fine if you can put up with its very idiosyncratic Lisp. Lots of people can't.

   Stalin is useful if your code is 100% debugged and bummed for performance and *still* runs too slowly in Gambit.

   Bigloo lacks core features of Schemes, but if you don't need them (and lots of people's code does not), then it's a fine implementation.

   I've used Spock and Biwa, but not seriously. Unfortunately, neither one works in Rhino or other command-line JavaScript engines, which means I don't get to use them in my test suite.

   My personal favorites for development are Chicken and Chibi. I'm now using a style that will wind up delivering both a Chibi library and a Chicken egg.

   [My test suite](http://trac.sacrideo.us/wg/wiki/ImplementationContrasts); [on beyond zebra](http://community.schemewiki.org/?scheme-faq-standards#H-1rl08mk) (though still not complete).

9. [Janne](http://janneinosaka.blogspot.com) says:[8 January 2013 4:12 AM](#e42f25a479d95f17ec3f0036061a01e615254aad)

   While we're on the subject: is there any plan to start making external libraries such as Cairo work in Guile 2.x? As far as I can tell, no such external libs are available or working, at least not in Ubuntu. I like Guile, but it's very limited without any external functionality.

10. [Moritz Heidkamp](http://ceaude.twoticketsplease.de/) says:[8 January 2013 9:42 AM](#7e27508928e7f17ce02d1606741eadc0f0aae719)

    As mentioned on IRC already, I really like this post and will gladly point people to it in the future!

    Regarding JS Schemes I once compiled this survey of implementations which dearly needs an update but may serve some purpose in its current state anyway: http://ceaude.twoticketsplease.de/js-lisps.html

11. John says:[8 January 2013 9:48 AM](#6ee0ffa7b92b93b45f5d4bcb5e15ab4f8351966d)

    Also to mention s7 (https://ccrma.stanford.edu/software/snd/snd/s7.html) which is extremely fast interpreted scheme meant for embedding. Author is claiming it is almost fast as guile...

12. [wingo](http://wingolog.org/) says:[8 January 2013 2:00 PM](#0e7c912f486a956f01c1c90c7ec3e27d426e0a48)

    Janne: [guile-cairo](http://www.nongnu.org/guile-cairo/) 1.9.91 works fine with Guile 2.0, as does guile-gnome and guile-clutter. Perhaps I should make a new guile-cairo release though!

13. [Janne](http://janneinosaka.blogspot.com) says:[8 January 2013 2:21 PM](#f9359276db347c16fb68ac0333fa1ac5fa7c5c07)

    Ahh - it's the Ubuntu repositories that are way out of date I think. Guile-cairo there is 1.4 something...

14. Patrick Useldinger says:[8 January 2013 7:27 PM](#891c167368e678f8ae1de1b7219a22badb6181c3)

    You don't say anything about operating systems... I guess that Racket is the only simple and convenient choice for Windows users.

    Chibi and Chicken seem to be the best choice if you value support for the upcoming Scheme standard R7RS.

    Performance-wise, Bigloo seems to be the leader among the active implementations.

    The most active communities seem to be Chicken and Racket. All in all I'd suggest one of these 2 implementations, unless you have very good reasons to chose something else.

15. Jacob says:[9 January 2013 6:38 AM](#8f717e3a8dfccfd866b019a9dd1e288452a248e0)

    To be fair, pretty much any Scheme implementation should be fine for working through SICP examples. My school had used SCM and MIT Scheme in the past. Recently, I wanted to work though some SICP examples and exercises for the metacircular evaluator again, so I used whatever was available in my Cygwin install. I copied and pasted code into Guile 1.x and SCSH, and both worked flawlessly. Guile 2.x also works, but I am very disappointed with its newbie unfriendly defaults, such as starting the debugger every time you make a typo in the REPL shell. I'd think that displaying a friendly message "variable not defined" would have been enough. It took half hour to find out how to turn that off.

16. [jiyinyiyong](http://blog.jiyinyiyong.info) says:[10 January 2013 11:42 AM](#07c57c24f45b294226d36fc2a853b36dde6b1e6a)

    To be honest, I don't think Guile's docs is good enough. It's really tough for a beginner who writes language like JS or Python to read a text only format of Guile manual, while JS has docs on MDN and Lua has tutorials in PDF and they both look nice. And as a beginner this is what I concerned most in the past year. A nice documents like ANSI Common Lisp? http://acl.readthedocs.org/en/latest/ No, there isn't.

17. [Grant Rettke](http://www.wisdomandwonder.com) says:[12 January 2013 9:23 PM](#f5b2dd11fc3d75133cb06e3dfccc5c6771ffbb2b)

    Geiser also works well with Geiser.

18. Jonathan says:[24 January 2013 2:07 PM](#d3a9b088d7de70d4e39eb147400d9d55bd2c96d0)

    Don't know whether it's worth posting here but wingolog blog has been mentioned in the list of programming and software development blogs here-> http://www.talkora.com/technology/List-of-programming-and-software-development-blogs\_105 (look for entry #36 in the list)

19. dca says:[28 February 2013 7:02 PM](#c2d53acbbd09d69a5f90b59ab309b915e6f3a6b3)

    Which scheme implementations offer the richest set of libraries?

20. [adamo](http://blog.postmaster.gr) says:[29 March 2013 10:15 AM](#87ac4882f4307d6ff64a742ee81a18eb44f72cad)

    No tinyscheme reference?

21. scape says:[16 March 2014 1:22 PM](#47f221ba40b2c99abf35ea19e3449d6def43754d)

    How would you rate racket vs guile now, a year later? Does racket not offer interactive Dev in emacs, I thought it did with geiser. It also can work closely with C APIs. What are your thoughts? I also notice you don't mention Clojure, while not scheme it is pretty fantastic as a modern take on lisp.

22. [John Cowan](http://www.ccil.org/~cowan) says:[17 March 2014 9:09 PM](#d4150e2d8f5f765d1f503e8195a5acb6dff0a6c2)

    Chibi is a little larger than TinyScheme, but it is much faster and more complete. Furthermore, you can set config options to make it smaller.

23. Nick says:[26 December 2014 10:33 AM](#f98412f69cde388b518d6d299b1f37fd28f19e37)

    This discussion persuaded me not to waste my time learning Scheme.

    "since Scheme is so minimal, you begin to rely on extensions that are only present in your implementation ... "

    Riiight. So you implement and debug something in Scheme, and years later, you need to move it to some other implementation, maybe because the implementation you picked to start with died. Guess what, you basically have to rewrite it!

    No thanks.

24. Linus says:[21 August 2015 6:43 AM](#9178dbc2c5100c2727e9011359009b88bb15e96c)

    Hi Nick!

    There is no need to be scared of that. There are some schemes that have been under development for a really long time, and they won't just disappear. A long time ago (2001) I wrote an automounter with dependency checking. Just dug out the old sources and spent about 15 minutes making it run on my raspberry pi using guile. The only thing needed porting was the parsing of the commands used to get available disks. Other than that it was just very minor things.

25. [John Cowan](http://www.ccil.org/~cowan) says:[20 November 2015 11:28 PM](#d0f6cf319967b1d606fe4f65a7c41ca8a8dedd4c)

    Nick: Scheme and Lisp are in practice so flexible that you can easily introduce a shim layer of procedures or macros or both to keep old code running essentially forever. Obsolescence that would be a disaster in any other language (syntactic changes particularly) is easy to fix up in Lisps.

26. Reuben Thomas says:[4 January 2016 9:35 PM](#df7bbe215ff22cdce25eb5158bcd15bccf4b01d6)

    Typo: "langauge" → "language"

27. bif says:[12 January 2016 2:55 AM](#64c72f975dddf340df878c34e1827fd4e17c04f0)

    All the Scheme implementations are terrible... maybe it's because the self-appointed RNRS standards committee comes out with a new backwards-incompatible standard every few years wasting implementors' time chasing it and breaking existing programs preventing users and implementors from investing in Scheme since it's such a moving target. Possibly the best language ever, driven into the ground. Why are we even following these people?

    e.g. there was a God-like compiler called Stalin that crushed all competition ("Stalin optimizes brutally" har har) - what happened to it? it's obsolete because it didn't keep up with the standards (no doubt because the guy behind it has better things to do), so all incremental improvements to it by other people would be pointless because no one uses it since it didn't keep up with the moving standard, and no one wants to do the boring work of updating it (which would also be difficult since it was so advanced). And yet there is a new litter of slow buggy Schemes to choose from.

    If Scheme exists for academic purposes - to implement Scheme mostly and to teach the occasional class, then fine, but why a new standard every few years that breaks programs that used the older standards?

28. dbohdan says:[31 January 2016 4:07 PM](#3c5cfc09f10368fd6d6c77dcc12922dc4eb2039d)

    Regarding John's 2013 comment: Chibi Scheme has since included a static library build target ("libchibi-scheme.a") in its Makefile.

29. [John Cowan](http://www.ccil.org/~cowan) says:[27 April 2016 9:22 PM](#0437d128894b8ce0e05df8b30af428ec00ba1b42)

    What bif says is not true. But I would say that, wouldn't I?

    We can now add Chez to the list of systems producing fast executables, though you have to ship Petite Chez with your program.

30. [John Cowan](http://vrici.lojban.org/~cowan) says:[17 September 2018 8:25 PM](#a60bb6e8c11a5dfedf9e96edc87d9a729b26a55d)

    Jakheg: No, it doesn't. I meant that you have to *at least* ship Petite with your compiled code in order to have a working executable, unlike the situation with Chicken and Gambit.

Comments are closed.
