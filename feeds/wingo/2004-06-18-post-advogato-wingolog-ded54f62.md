---
title: post-advogato. — wingolog
url: https://wingolog.org/archives/2004/06/18/post-advogato
published: "2004-06-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/06/18/post-advogato
---

# post-advogato. — wingolog

## [post-advogato.](/archives/2004/06/18/post-advogato)

18 June 2004 4:39 PM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

**it's nice to be back**

Indeed, it's nice. The loss of [advogato](http://advogato.org/) left me without an outlet for my journalling. Why I can't journal in private, I don't know. But I'm back, and it feels good.

**sxml**

Oleg Kiselyov wrote a SAX XML parser for scheme, [SSAX](http://okmij.org/ftp/Scheme/xml.html). The really interesting thing about it is that as it parses, it performs a tree-fold over the abstract syntax tree of the document. The user supplies the fold procedures, which are expected to modify a seed that they receive in their arguments. As the state of the parser is reflected in the scheme stack itself, the user is freed from maintaining her own stack of open elements. It's simple to program for, and the idea of a seed being threaded through the document is very beautiful. My mind was opening as I read his code.

**texinfo**

As I mentioned before on advogato, I am of the opinion that texinfo is a wonderful document-writing format, and also that info sucks. I had written one texinfo-processing framework, but its sexp realization was a kludge, and the parser itself wasn't too pretty either. It didn't even chunk the content into paragraphs when necessary, rather leaving users to interpret raw newlines.

After reading Oleg's code, I decided that it was time to do the job nicely. Also, this would bring texinfo into the realm of XML processing, an area full of activity. It took me about a month or so, on top of doing a lot of work at school, but that's what I did. I'm happy now. I've never written a beautiful parser before. (It can be found in the guile-lib--wingo under my arch archive, [wingo@pobox.com--2004-main](http://ambient.2y.net/wingo/arch/wingo@pobox.com--2004-main/).)

**profiling in scheme**

After seeing the performance of my new parser, especially compared to the old one, I was worried that I had failed. Of course it analyzed a lot more of the semantic content of the document, but still, it ran a noticeable three times slower.

Rather than give up, I started looking for a good profiler for guile. I found one in statprof, [languishing in guile CVS](http://lists.gnu.org/archive/html/guile-user/2001-11/msg00016.html). Statprof is a statistical profiler, like [qprof](http://www.hpl.hp.com/research/linux/qprof/), except it reads guile debug stack frames rather than the C stack frames. It is rather good at profiling scheme code, even offering the ability to instrument function calls. Best of all, it's written in pure scheme, which is fun to hack on. I added the ability to not instrument function calls (it's quite expensive in a functional language!), added time within the function itself in addition to the cumulative (with subroutines) time, and added a macro for easy profiling of code. It was fun hacking, and it helped me to make the new parser faster than the old.

The big win was to call functions that were implemented in C. Even though they might not have been as optimal, algorithmically speaking, the fact that they were written in C made them a lot faster. I wonder what the balance would be in a scheme that supports compilation.

**totalizing frameworks**

I caught the affliction of object-oriented programming a few years ago. I wanted objects everywhere. I couldn't bear to program any other way---it just felt dirty.

Then I learned Scheme. At first, I programmed in much the same way---always using slot-set! on GOOPS objects---but as I read more of the Scheme / Lisp literature, I began to feel dirty again. This time, my affliction was functional programming. It started with SICP teaching me about referential transparency, and is just now reaching its peak.

Some things are taking the edge off my fervor, though. Paul Graham's [On Lisp](http://www.paulgraham.com/onlisp.html) supports functional interfaces, even if within a procedure or set of procedures some imperative programming is used (reverse!, for instance). Imperative programming is what adds semantic spice to call/cc and closures with shared state, and hey, I still use objects these days, but in more appropriate circumstances.

At the same time, I am falling strongly towards a more powerful affliction: everything-in-scheme. Despite the complexity of wordpress, I was tempted to roll a blog myself just so that it could be implemented in scheme. But that would lead me to do all sorts of stupid things: mod\_guile, database access, etc. Then I would never release soundscrape!

**mom got drunk, and dad got drunk...**

at our christmas party... we were drinkin champagne punch and homemade eggnog...

Rose up some hell last weekend in the village, serenading various bars with Robert Earle Keen Jr. They had all closed, and the shopkeepers had gone to sleep inside their stores. Nobody opened up for us, though! One woman who runs the family's bar there said she thought we were botsotsos (thieves). Which thieves rob people with guitars?

**and finally**

My code looks stupid after reading *On Lisp*. Damn.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)

Comments are closed.
