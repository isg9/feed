---
title: a brief history of guile — wingolog
url: https://wingolog.org/archives/2009/01/07/a-brief-history-of-guile
published: "2009-01-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/01/07/a-brief-history-of-guile
---

# a brief history of guile — wingolog

## [a brief history of guile](/archives/2009/01/07/a-brief-history-of-guile)

7 January 2009 9:01 PM

- [guile](/tags/guile)
- [history](/tags/history)
- [emacs](/tags/emacs)
- [scheme](/tags/scheme)

I'd like to share with you all here some writing that I did last month for [Guile](http://www.gnu.org/software/guile/)'s manual. It runs through the 15-year history of Guile Scheme, re-visions a commonly misunderstood episode, and gosh darnit I just really like "the emacs thesis".

You can find the original [here](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob_plain;f=doc/ref/history.texi;hb=vm).

\\* \\* \\*

#### The Emacs Thesis

The story of Guile is the story of bringing the development experience of Emacs to the mass of programs on a GNU system.

Emacs, when it was first created in its GNU form in 1984, was a new take on the problem of "how to make a program". The Emacs thesis is that it is delightful to create composite programs based on an orthogonal kernel written in a low-level language together with a powerful, high-level extension language.

Extension languages foster extensible programs, programs which adapt readily to different users and to changing times. Proof of this can be seen in Emacs' current and continued existence, spanning more than a quarter-century.

Besides providing for modification of a program by others, extension languages are good for *intension* as well. Programs built in "the Emacs way" are pleasurable and easy for their authors to flesh out with the features that they need.

After the Emacs experience was appreciated more widely, a number of hackers started to consider how to spread this experience to the rest of the GNU system. It was clear that the easiest way to Emacsify a program would be to embed a shared language implementation into it.

#### Early Days

Tom Lord was the first to fully concentrate his efforts on an embeddable language runtime, which he named "GEL", the GNU Extension Language.

GEL was the product of converting SCM, Aubrey Jaffer's implementation of Scheme, into something more appropriate to embedding as a library. (SCM was itself based on an implementation by George Carrette, SIOD).

Lord managed to convince Richard Stallman to dub GEL the official extension language for the GNU project. It was a natural fit, given that Scheme was a cleaner, more modern Lisp than Emacs Lisp. Part of the argument was that eventually when GEL became more capable, it could gain the ability to execute other languages, especially Emacs Lisp.

Due to a naming conflict with another programming language, Jim Blandy suggested a new name for GEL: "Guile". Besides being a recursive acroymn, "Guile" craftily follows the naming of its ancestors, "Planner", "Conniver", and "Schemer". (The latter was truncated to "Scheme" due to a 6-character file name limit on an old operating system.) Finally, "Guile" suggests "guy-ell", or "Guy L. Steele", who, together with Gerald Sussman, originally discovered Scheme.

Around the same time that Guile (then GEL) was readying itself for public release, another extension language was gaining in popularity, Tcl. Many developers found advantages in Tcl because of its shell-like syntax and its well-developed graphical widgets library, Tk. Also, at the time there was a large marketing push promoting Tcl as a "universal extension language".

Richard Stallman, as the primary author of GNU Emacs, had a particular vision of what extension languages should be, and Tcl did not seem to him to be as capable as Emacs Lisp. He posted a criticism to the comp.lang.tcl newsgroup, sparking one of the internet's legendary flamewars. As part of these discussions, retrospectively dubbed the "Tcl Wars", he announced the Free Software Foundation's intent to promote Guile as the extension language for the GNU project.

It is a common misconception that Guile was created as a reaction to Tcl. While it is true that the public announcement of Guile happened at the same time as the "Tcl wars", Guile was created out of a condition that existed outside the polemic. Indeed, the need for a powerful language to bridge the gap between extension of existing applications and a more fully dynamic programming environment is still with us today.

#### A Scheme of Many Mantainers

Surveying the field, it seems that Scheme implementations correspond with their maintainers on an N-to-1 relationship. That is to say, that those people that implement Schemes might do so on a number of occasions, but that the lifetime of a given Scheme is tied to the maintainership of one individual.

Guile is atypical in this regard.

Tom Lord maintaned Guile for its first year and a half or so, corresponding to the end of 1994 through the middle of 1996. The releases made in this time constitute an arc from SCM as a standalone program to Guile as a reusable, embeddable library, but passing through a explosion of features: embedded Tcl and Tk, a toolchain for compiling and disassembling Java, addition of a C-like syntax, creation of a module system, and a start at a rich POSIX interface.

Only some of those features remain in Guile. There were ongoing tensions between providing a small, embeddable language, and one which had all of the features (e.g. a graphical toolkit) that a modern Emacs might need. In the end, as Guile gained in uptake, the development team decided to focus on depth, documentation and orthogonality rather than on breadth. This has been the focus of Guile ever since, although there is a wide range of third-party libraries for Guile.

Jim Blandy presided over that period of stabilization, in the three years until the end of 1999, when he too moved on to other projects. Since then, Guile has had a group maintainership. The first group was Maciej Stachowiak, Mikael Djurfeldt, and Marius Vollmer, with Vollmer staying on the longest. By late 2007, Vollmer had mostly moved on to other things, so Neil Jerram and Ludovic Court s stepped up to take on the primary maintenance responsibility.

Of course, a large part of the actual work on Guile has come from other contributors too numerous to mention, but without whom the world would be a poorer place.

#### A Timeline of Selected Guile Releases

guile-i --- 4 February 1995

SCM, turned into a library.

guile-ii --- 6 April 1995

A low-level module system was added. Tcl/Tk support was added, allowing extension of Scheme by Tcl or vice versa. POSIX support was improved, and there was an experimental stab at Java integration.

guile-iii --- 18 August 1995

The C-like syntax, ctax, was improved, but mostly this release featured a start at the task of breaking Guile into pieces.

1.0 --- 5 January 1997`#f` was distinguished from `'()`. User-level, cooperative multi-threading was added. Source-level debugging became more useful, and programmer's and user's manuals were begun. The module system gained a high-level interface, which is still used today in more or less the same form.

1.1 --- 16 May 19971.2 --- 24 June 1997

Support for Tcl/Tk and ctax were split off as separate packages, and have remained there since. Guile became more compatible with SCSH, and more useful as a UNIX scripting language. Libguile can now be built as a shared library, and third-party extensions written in C became loadable via dynamic linking.

1.3.0 --- 19 October 1998

Command-line editing became much more pleasant through the use of the readline library. The initial support for internationalization via multi-byte strings was removed, and has yet to be added back, though UTF-8 hacks are common. Modules gained the ability to have custom expanders, which is still used for syntax-case macros. Ports have better support for file descriptors, and fluids were added.

1.3.2 --- 20 August 19991.3.4 --- 25 September 19991.4 --- 21 June 2000

A long list of lispy features were added: hooks, Common Lisp's `format`, optional and keyword procedure arguments, `getopt-long`, sorting, random numbers, and many other fixes and enhancements. Guile now has an interactive debugger, interactive help, and gives better backtraces.

1.6 --- 6 September 2002

Guile gained support for the R5RS standard, and added a number of SRFI modules. The module system was expanded with programmatic support for identifier selection and renaming. The GOOPS object system was merged into Guile core.

1.8 --- 20 February 2006

Guile's arbitrary-precision arithmetic switched to use the GMP library, and added support for exact rationals. Guile's embedded user-space threading was removed in favor of POSIX pre-emptive threads, providing true multiprocessing. Gettext support was added, and Guile's C API was cleaned up and orthogonalized in a massive way.

2.0 --- thus far, only unstable snapshots available

A virtual machine was added to Guile, along with the associated compiler and toolchain. Support for internationalization was added. Running Guile instances became controllable and debuggable from within Emacs, via GDS. GDS was backported to 1.8.5. An SRFI-18 interface to multithreading was added, including thread cancellation.

#### Status, or: Your Help Needed

Guile has achieved much of what it set out to achieve, but there is much remaining to do.

There is still the old problem of bringing existing applications into a more Emacs-like experience. Guile has had some successes in this respect, but still most applications in the GNU system are without Guile integration.

Getting Guile to those applications takes an investment, the "hacktivation energy" needed to wire Guile into a program that only pays off once it is good enough to enable new kinds of behavior. This would be a great way for new hackers to contribute: take an application that you use and that you know well, think of something that it can't yet do, and figure out a way to integrate Guile and implement that task in Guile.

With time, perhaps this exposure can reverse itself, whereby programs can run under Guile instead of vice versa, eventually resulting in the Emacsification of the entire GNU system. Indeed, this is the reason for the naming of the many Guile modules that live in the `ice-9` namespace, a nod to the fictional substance in Kurt Vonnegut's novel, Cat's Cradle, capable of acting as a seed crystal to crystallize the mass of software.

Implicit to this whole discussion is the idea that dynamic languages are somehow better than languages like C. While languages like C have their place, Guile's take on this question is that yes, Scheme is more expressive than C, and more fun to write. This realization carries an imperative with it to write as much code in Scheme as possible rather than in other languages.

These days it is possible to write extensible applications almost entirely from high-level languages, through byte-code and native compilation, speed gains in the underlying hardware, and foreign call interfaces in the high-level language. Smalltalk systems are like this, as are Common Lisp-based systems. While there already are a number of pure-Guile applications out there, users still need to drop down to C for some tasks: interfacing to system libraries that don't have prebuilt Guile interfaces, and for some tasks requiring high performance.

The addition of the virtual machine in Guile 2.0, together with the compiler infrastructure, should go a long way to addressing the speed issues. But there is much optimization to be done. Interested contributors will find lots of delightful low-hanging fruit, from simple profile-driven optimization to hacking a just-in-time compiler from VM bytecode to native code.

Still, even with an all-Guile application, sometimes you want to provide an opportunity for users to extend your program from a language with a syntax that is closer to C, or to Python. Another interesting idea to consider is compiling e.g. Python to Guile. It's not that far-fetched of an idea: see for example IronPython or JRuby.

And then there's Emacs itself. Though there is a somewhat-working Emacs Lisp translator for Guile, it cannot yet execute all of Emacs Lisp. A serious integration of Guile with Emacs would replace the Elisp virtual machine with Guile, and provide the necessary C shims so that Guile could emulate Emacs' C API. This would give lots of exciting things to Emacs: native threads, a real object system, more sophisticated types, cleaner syntax, and access to all of the Guile extensions.

Finally, there is another axis of crystallization, the axis between different Scheme implementations. Guile does not yet support the latest Scheme standard, R6RS, and should do so. Like all standards, R6RS is imperfect, but supporting it will allow more code to run on Guile without modification, and will allow Guile hackers to produce code compatible with other schemes. Help in this regard would be much appreciated.

### coda

(This part's not in the document. If you've gotten here, thanks for reading!)

The closest thing to Emacs these days is the web browser, and who do we find there: Jim Blandy's now at Mozilla working on ActionMonkey, and Maciej Stachowiak is at Apple hacking Webkit. Neat. Sweet, even!

But to me there's a weakness in JavaScript, that it's a kind of watered-down Scheme. You can't buy good wine in the supermarket; you have to go closer to the producer. Similarly if you want to hack something really delightful, go to what the hackers hacked back before they had mortgages -- and what they [still hack on](http://www.red-bean.com/trac/minor/) in their free time.

This document was made with Guile, and [served](//wingolog.org/software/tekuti/) with it too.

```
guile <xml
 (stexi->shtml
  (call-with-input-file "history.texi" texi-fragment->stexi)))
EOF

```

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)
- [wikipedia & guile](/archives/2009/10/01/wikipedia-guile)
- [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 15 responses

1. Boris says:[7 January 2009 11:52 PM](#bc93f1ee78f7792ae71748a80e70fe5c9eaeb273)

   That was a really interesting read, thank you.

   I have only begun to learn Scheme, and for that I'm using Guile, so this write up caught my attention.

   Your post said that one of the original plans for Guile was to provide a sort of common infrastructure for extensibility in Free Software and that the community should strive for that goal, but hasn't that niche already been filled by Python?

   Python is seemingly everywhere, as the high-level language of choice for applications programming and as an extensions language, what are your plans to convince hackers that Scheme and Guile are the way forward instead of Python and its interpreter?

2. [Janne](http://janneinosaka.blogspot.com) says:[8 January 2009 1:40 AM](#a48eb365a35f28bb33a556d5d47bb32cd0940b53)

   I think that Python - and Ruby and Perl, as well as Java, C# and so on - are too heavyweight. They're full-blown environments in their own right. For an application scripting engine you want something without any heavy library baggage; it should use whatever support libs and functionality its underlying application is using, not bring in any of its own. For that kind of thing, JS and Scheme are both good choices.

   With that said, I'm a decent programmer, and I really \_want\_ to like Scheme. I do grasp the basics, I have no problem with tail recursion to express loops and so on. But then I stumble into more advanced stuff, like the nightmare of object representation, with a morass of hard to read, hard to grasp syntax and internal interpreter gunk exposed everywhere, and I give up.

   I realize that I simply do not have the energy to penetrate all that and actually become productive in the language. In most languages you learn the basic syntax and that's it, really. Even very advanced things are still expressed using the same basic syntactical and semantic constructions. Ruby, for instance, still looks and feels the same even when you're doing some pretty intense (and rarely needed) metaprogramming. But for Scheme (and worse, Lisp), the whole language structure keeps changing right under my feet. Your series of posts on object orientation under Guile has frankly had me completely lost; things I thought I understood about the language turn out to be completely different (maybe; I'm too confused to be sure) and has made me give up.

   I'm sure a good programmer can get it quite easily. But for an extension language, what you need is for middling, so-so, jobbing programmers like me to be able to grasp it and make use of it or it'll never see wide acceptance. I'm not doing programming for it's own sake, but because I'm trying to get my real work done and that involves writing code. I have neither the inclination nor the luxury of time to really delve down and spend three or six months learning a complex language in the hope that I'll be more productive - and happier, and a lottery winner, with a pony! - down the road.

   Sorry, long rant. I really do want to like Scheme. Right now, I just can't. JS is ugly but it does have the benefit of being easy to understand.

3. Justin says:[8 January 2009 3:15 AM](#2a353a86dc04d2847c7f762dbe704d9fdb508436)

   Excellent post, I really enjoyed reading it.

   I have fallen in love with many languages from Erlang to haskell but no matter what I do I always seem to come back to Lisp.

   I have never specifically used Guile but if I do need to embed an extension language it will certainly be my first port of call.

4. [Mathias Hasselmann](http://taschenorakel.de/mathias/) says:[8 January 2009 4:17 AM](#b57eaec292e2752201123b9abc197a1d1d4ec6e4)

   Excellent post.

   There is just one problem: Guile is a Scheme dialect an therefore inherits this horrible, barely readable syntax. That's the reason why it failed. Only few of the post 1970 hackers wants to do Scheme, if they can have Javascript, Python, Ruby, ... which all have a much more readable syntax. Well, and code is about reading.

5. Xan says:[8 January 2009 2:59 PM](#3a0dda639044072bf78447703c39683f3e46a0bb)

   @Janne: how does the 'syntax' in Common Lisp change when dealing with object orientation? Generic functions use exactly the same syntax than non-generic functions, so it's even more uniform than other languages where the syntax for method invocation is different than common functions (!?).

   Also the idea that you need months to pick up Scheme is a bit weird. Lisp is one of the languages with less syntax in existence, and you can learn pretty much all you need in that regard in an afternoon without any exaggeration. Now, becoming \*good\* with Lisp, that's a different matter, but I don't think it's different than with any other language.

6. Paul Prescod says:[8 January 2009 5:30 PM](#d9aa929e2843b76213dd6bf1c1c957f1244d3240)

   I could not disagree more strongly with Janne's opinion that "r an application scripting engine you want something without any heavy library baggage;"

   If you look at the libraries in Python, the vast majority of them help in integrating components together. Application extensions are VERY FREQUENTLY about integrating external systems.

   Here's an example from my past, back when PowerPoint was by far the dominant spreadsheet:

   http://oreilly.com/pub/a/oreilly/frank/pyt

   "My favorite demo, though, was Paul Prescod's conversion program. Many of you will recognize Paul as an XML authority and supporter. It irks him when conferences require him to deliver his slides in PowerPoint, an appearance-heavy presentation product that is anathema to semantic-loving XML experts like Paul. Paul built, and demonstrated, a conversion program that allows him to put together his presentations in XML and then use Python and COM to convert it to PowerPoint. "I'll deliver my talks in PowerPoint, but I won't use it," Paul bragged.

7. Paul Prescod says:[8 January 2009 7:07 PM](#fff75ee758cb30d1ee2f7a01608ca7140f731e69)

   Janne, I also find it incomprehensible that you claim that Ruby has a more consistent syntax than Scheme once you get into OO. If I recall correctly, Ruby has distinct syntax for methods, class methods and functions. It also has distinct syntax for instance variables and class variables (as opposed to "class-level instance attributes").

8. [wingo](http://wingolog.org/) says:[8 January 2009 9:52 PM](#117d6c591e1776a2efb10be83274b6903320b3d5)

   Thanks all for reading, and the comments.

   @Boris: I don't think that Python has really filled that niche -- which emacs-like programs do you know that use python as their extension language? Not that Python is a bad language, it's just not malleable enough to allow Emacs.

   @Janne: In many ways it's enough to have given the Lisp family an honest try. I know a number of really good hackers who say that their brains just don't map natively to Lisp, and that's fine. *Queremos un mundo donde quepan muchos mundos*, as say the Zapatistas.

   @Matthias: Well it certainly seems that Guile failed to filter out a snarky comment about it! I'll have to tune that AI transducer, or something.

9. [Gregg Williams](http://www.InfoML.org) says:[9 January 2009 4:26 AM](#5a6c5dc696bb425a282a0eb54c6f10ac72eff6ca)

   Hi-- I'm really interested in seeing http://oreilly.com/pub/a/oreilly/frank/pyt, but when I try it I get a 500 Internal Server Error.

   Paul could you check the URL? Many thanks.

   --Gregg Williams

   section editor for the special LISP issue of Byte magazine

10. Paul Emsley says:[11 January 2009 5:21 PM](#9f7900ca992dccfcf5b223658e6f81e2e50ad7e5)

    Nice read - thanks.

    I have a comment/question: Thomas Bushnell's Guile Python Compiler. Do you know about it? (It's not been worked on for a while...)

    http://savannah.gnu.org/projects/gpc

11. [wingo](http://wingolog.org/) says:[11 January 2009 8:58 PM](#7e5a44ec2e3a1356012ed6c2558ae08e92708a82)

    @Paul: No I hadn't seen that. I'll take a look, thanks!

12. fnc says:[20 January 2009 3:56 AM](#da246edd4b809f95b1243c6966a328489d6cd3d3)

    I like Scheme but I think it is too fragmented. Only now with R6RS they have a standard module system and I think they never will have a standard object system. Nowadays the most adequate language and implemention for this domain (application scripting) is probably Lua. It has a small core language, a fast and small standard implementation and reasonable number of libraries and applications (less than Perl or Python, but more than Guile).

13. eug says:[20 January 2009 4:16 PM](#27473c6c6261d2a0baa7c23c82ea377f3cbab08d)

    IMHO

    When writing an extension...

    Most of the time I implement the logic/algorithm in C++

    and the "glue" part - in some scripting language (Bash, MS-batch)

    Scheme is neither simple in syntax nor fast at runtime

14. namekuseijin says:[4 May 2009 9:45 AM](#f9c166cfa48a24071e2c6a6626370c4d506847b3)

    Very interesting article. I knew the general history of Guile and its relation to the GNU project, but not to this extent. And while it didn't quite get to be the preferred scripting way in GNU/Linux systems, it's still good to have any way at all to do system scripting in Scheme besides scsh.

    @Mathias: what syntax?

    @fnc: C doesn't have OO either and it never hampered it. I consider data structures like records, vectors and lists far more important.

15. [arnuld](http://LispMachine.wordpress.com) says:[3 June 2011 11:25 AM](#0f543da96209a04a772e476608127feb81b0d958)

    Very good article. It was a lot of fun reading Guile History as written by you and I am sure there will be a lot of fun in re-reading too :)

    Came across it while I was searching for whether to learn Lua or Guile for scritpting and I see both are quite different. Guile is an extension language while Lua purely a scripting langauge. Personally I don't like Scheme (I like Common Lisp) but this writing of yours almost convinced me to look into Guile Programming once :)

Comments are closed.
