---
title: 'technicalities: interactive scientific computing #2 of 2, goldilocks languages'
url: https://graydon2.dreamwidth.org/189377.html
published: "2014-07-11T22:35:10Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:189377
---

# technicalities: interactive scientific computing #2 of 2, goldilocks languages

In my [previous post](http://graydon2.dreamwidth.org/3186.html) I discussed [Ousterhout's Dichotomy](https://en.wikipedia.org/wiki/Ousterhout's_dichotomy) at some length and suggested that it was ubiquitous in computing. In this post I will discuss some famililes of language that reject Outsterhout's Dichotomy, their history, and a new language called Julia.

As usual, this is a (very) long, boring post for people who don't care about languages, so I'll cut it for brevity.

![](http://venge.net/graydon/mccarthy-wrong.jpeg)

## Puritans

In the [previous post](http://graydon2.dreamwidth.org/3186.html) I described a certain *majority position* concerning languages. An orthodoxy, perhaps. That is: most languages -- certainly the earliest few -- were written by people who were, at the moment of writing, more interested in some practical non-language problem (counting a census, calculating artillery trajectories) than they were in the design of languages per se. Thus we wound up with languages characterized by some particular task they're "good at" -- because it's what they were designed to help automate -- and a bunch of tasks they are flatly *bad at*. The first "big" language, [Fortran](http://en.wikipedia.org/wiki/Adaptive_optimization), has these sort of hallmarks. Significantly, most such languages have a strong preference for one or another "ends of the spectrum" between the concerns of interactive human reasoning-with-code and the concerns of raw, batch-processing computation; and so each such language winds up paired with some code from a different language, from the other side of Ousterhout's Dichotomy.

However, for a long time there has been a *substantial dissenting minority*. There are languages and systems built by people who *reject* the notion that human interactivity and batch performance (or any other specific usage-domains for languages) are in inherent tension in language design; or else they believe that the tension is greatly overstated, and that *carefully designed* languages can be built that are "good enough" for all use cases. General enough, powerful enough, interactive enough, fast enough. And that the benefits of such "Goldilocks" languages greatly outweighs the price of maintaining Ousterhout-style hybrid systems built out of multiple languages.

To understand these people and their languages, it helps to know where they came from and what they've built along the way. Spoiler: it involves, to a large extent, **[Lisp](http://en.wikipedia.org/wiki/Lisp_%28programming_language%29)**.

And I'm going to moderate nit-picking comments about Lisp history pretty aggressively, because I have a point to make here and it's *not* to write the definitive history of Lisp.

[![Gentzen kalkulus](https://farm4.staticflickr.com/3296/3128026452_f450873ae7.jpg)](https://www.flickr.com/photos/21923568@N00/3128026452 "Gentzen kalkulus by Gábor Kovács, on Flickr")

## Symbolic computation, logic, Lisp

In the previous post I mentioned in passing a discipline within scientific computing called ["symbolic computation"](https://en.wikipedia.org/wiki/Symbolic_computation): computation at the level of indeterminate values rather than concrete numbers. This is the computerization of the disciplines of algebra and mathematical logic, in which one manipulates expressions according to rules for [rewriting](https://en.wikipedia.org/wiki/Rewrite_system), equality and logical implication, rather than manipulating numerical values based on arithmetic and tables of values. As I mentioned in the previous post, the two parallel forks of development here were ["computer algebra systems"](https://en.wikipedia.org/wiki/Computer_algebra_system) (CASs) for symbolic math, and "logical frameworks" or ["proof assistants"](https://en.wikipedia.org/wiki/Proof_assistant) for symbolic logic. In both cases the tools often have to do forms of heuristic *search* in "expression spaces", in some sense similar to the way numerical systems search in Euclidean "numeric spaces" for exact numerical solutions to equality or optimization problems. I've talked a little about logical frameworks before [in an older post as well](http://graydon2.dreamwidth.org/1839.html).

![](http://venge.net/graydon/logic-theory-machine.png)This sounds like a bit of a diversion, but the setting's important for what happens: the interesting history here is that in the process of *implementing* formal reasoning tools that manipulate symbolic expressions, researchers working on logical frameworks -- i.e. with background in mathematical logic -- have had a tendency to produce *implementation languages* along the way that are very ... *"tidy"*. Tidy in a way that befits a mathematical logician: orthogonal, minimal, utterly clear and unambiguous, defined more in terms of mathematical logic than machine concepts. Much clearer than other languages at the time, and much more amenable to reasoning about. The [original manual for the Logic Theory Machine and IPL (1956)](http://bitsavers.trailing-edge.com/pdf/rand/ipl/P-868_The_Logic_Theory_Machine_Jul56.pdf) makes it clear that the authors were deeply concerned that nothing sneak in to their implementation language that was some silly artifact of a machine; they wanted a language that they could hand-simulate the reasoning steps of, that they could be *certain* of the meaning of their programs. They were, after all, translating [Russel and Whitehead](https://en.wikipedia.org/wiki/Principia_Mathematica) into mechanical form!

![](http://venge.net/graydon/nqthm.png)IPL was the implementation layer for Logic Theory Machine, Lisp (1958) was IPL's descendant and the implementation layer of pretty much all the symbolic reasoning tools for the next 35 years (until Java), be they symbolic-logic focused or symbolic-arithmetic focused: systems like Macsyma (algebra, 1968) or NQTHM (logic, 1971) built directly in Lisp; systems like Axiom (algebra, 1965) or LCF (logic, 1972) building strongly typed higher level symbolic languages implemented in Lisp; in all cases, the family here was strongly rooted in *minimal, clear, symbolic Lisp culture*.

![](http://www.oughtred.org/mitmuseum/mit-lisp-machine-3478-72lg.jpg)Lisp's tidy definition also gave it a certain plausibility for fast implementation, or at least as an object of experimentation for people interested in clever language implementation. While its first implementation by was a slow interpreter, it quickly gained [incremental compilation](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-039.pdf) and eventually even dedicated hardware support. Moreover, because Lisp code was simple to manipulate and reason about, it was simple for students and academics to write compilers and program transformations in; Lisp and its progeny Scheme and ML quickly became platforms for [research in language implementation](http://library.readscheme.org/page1.html) that had few parallels.

(Possible exceptions might be the [Forth](https://en.wikipedia.org/wiki/Forth_(programming_language)) lineage or the [Oberon System](http://en.wikipedia.org/wiki/Oberon_%28operating_system%29), each with similar minimal, efficient, interactive compilers; there are [77 Schemes](http://community.schemewiki.org/?scheme-faq-standards#implementations), [a dozen MLs](http://en.wikipedia.org/wiki/Standard_ML#Implementations) and [37 Common Lisps](http://en.wikipedia.org/wiki/Common_Lisp#Implementations) vs. [126 Forths](http://wiki.forthfreak.net/index.cgi?ForthSystemsAlphabetical).)

![](http://venge.net/graydon/squeak.png)

![](http://upload.wikimedia.org/wikipedia/en/d/d2/Symbolics-document-examiner.png)

It's also worth reiterating: *Lisps excelled at interactivity*. All the major Lisps sport high quality REPLs and many contain sophisticated mixed-mode editors, browsers, debuggers and GUIs for inspecting, modifying, visualizing and searching the insides of running systems. The two gold standards for hypertextual, interactive, multimedia programming systems are still probably [Lisp Machine Genera (1982)](http://en.wikipedia.org/wiki/Genera_%28operating_system%29) and the [Smalltalk Workspace (1976)](http://en.wikipedia.org/wiki/Smalltalk). Today's IDEs spend millions of lines of code and consume gigabytes of memory in pursuit of interaction quality that is arguably not much advanced over things shipping 30+ years ago.

There has been a lot of running around in circles.

[![](http://venge.net/graydon/langpop.png)](http://langpop.com)

While I'm not trying to paint too rosy a picture here -- the Lisp Machine market never got out of its (very expensive) niche and Lisps have largely remained at the margins of mainstream computing -- it's worth understanding that *many Lisp people never accepted Ousterhout's Dichotomy*, nor did most of the people working with languages that were descendants of Lisps, such as the Smalltalks and MLs. They built languages that were reflective, dynamic, interactive, human-focused ... yet *also* implemented in *tolerably* low-level, fast, machine-code compilers (MacLisp, KCL/GCL, SBCL; Rabbit, Orbit, Gambit, Stalin; SML/NJ, Ocaml, MLton, Alice; Smalltalk80, Self). Some of these were static compilers, some dynamic. Some clever, some basic. But usually such Goldilocks languages were grown with native-code compilers of *some* sort from the get-go, and their developers and proponents argued strenuously that *no other language was necessary* to build a complete, performant, humane system.

No Ousterhout-style hybrids. One language, if it was good enough, should be enough.

## INTERMISSION

![](http://venge.net/graydon/elitist.png)

## OOP, The Expression Problem, Multimethods, costs, Dylan

Lisps (and later, Smalltalks) were also where a lot of the early research on ["Object Orientation"](http://en.wikipedia.org/wiki/Object-oriented_programming#History) came from. I don't much care about OOP from a philosophical perspective, but technologically it represents an early sort of grappling with [The Expression Problem](http://en.wikipedia.org/wiki/Expression_problem). At least initially -- well before it was named and studied -- this was the problem of adding new *data types* to an existing system without having to rewrite all your procedures to handle them. Later, as OOP matured, it became clear that the problem was symmetric and that (single-dispatch) OOP hadn't solved the opposite side of the problem: one could add new types easily; but adding new procedures caused you to have to change all your types. Therefore in the general formulation of the expression problem, each axis of extension -- types or procedures -- is treated as equally desirable. Sometimes you want to extend one, sometimes the other. Languages ought to support either, without causing you to rewrite too much code. Check [these slides](http://userpages.uni-koblenz.de/~laemmel/paradigms1011/resources/pdf/xproblem.pdf) out if it's still not clear what I'm talking about here, it's a bit subtle.

![](http://upload.wikimedia.org/wikipedia/en/3/30/The_Art_of_the_Metaobject_Protocol_cover.jpg)

I said above that OOP in its simplest "single-dispatch" form doesn't solve the expression problem terribly well: it solves about half of it. But in an other, more general form that emerged around the middle of OOP's initial hype phase, a special sort of OOP *does much better*. This form is called "multimethods" or [**"multiple dispatch"**](http://en.wikipedia.org/wiki/Multiple_dispatch). I will not say that multimethods "solve" the expression problem, because I think there are are always going to be corner cases. But they do a surprisingly good job. The rest of this post will look at them in terms of the historical space of Lisps, our willingness to pay for certain language technology, Dylan, and finally (finally!) Julia.

![](http://venge.net/graydon/cm200-6sm.jpg)Multimethods are subtle enough as a programming concept that I won't do a good job explaining them here in detail. The linked page above is helpful, as are links from it, as will be a page I'm going to link to down below about Julia. Suffice to say: they let you add data types and procedures to an existing program, without bias to one or the other, with relatively similar amounts of (programmer) work, and that work is linear with the amount you add, not the existing system size. So they're *great*, a great language feature. They were introduced most convincingly in the [Common Lisp Object System (CLOS)](http://en.wikipedia.org/wiki/Common_Lisp_Object_System) in the late 1980s, unfortunately right around the time at which the final gasps of the Lisp AI market in the [second AI winter](http://en.wikipedia.org/wiki/AI_winter#The_setbacks_of_the_late_.2780s_and_early_.2790s). The idea was, at any rate, probably overengineering at the time and certainly considered (like many Lisp features) *too expensive* in terms of CPU cycles spent to warrant its use.

Many of Lisp's failures in the market ultimately come down to this sort of thing: the "Goldilocks" perspective on languages involves a willingness to trade *some* CPU and memory for features that improve programmer comfort, interactivity, safety, reflection, debugging simplicity and so forth. During much of Lisp's life, too few people were willing to pay those costs: it was too often seen as "too slow to be worth it". Its defenders, naturally, disagree.

![](http://venge.net/graydon/dylan.png)One of the last major languages to explore the feature-space of multimethods was a delightful research project out of [Apple's Advanced Technology Group](http://en.wikipedia.org/wiki/Advanced_Technology_Group_%28Apple%29) called [Dylan](http://en.wikipedia.org/wiki/Dylan_%28programming_language%29). The project started as part of the ill-fated [Newton PDA](http://en.wikipedia.org/wiki/Newton_%28platform%29) experiment (back when [tablet computers were science fiction devices](http://en.memory-alpha.org/wiki/PADD) we saw on [STTNG](http://en.wikipedia.org/wiki/Star_Trek:_The_Next_Generation)).

Dylan was a great language. It had many excellent Lisp-family designers working on it, it was a tasteful blend of Algol-ish syntax and Lisp-ish expression uniformity, it traded performance and dynamism smoothly across a mixture of compilation modes, it leaned heavily on multimethods and was pleasantly extensible while remaining relatively comprehensible, it performed well. Many language nerds were excited. But its fate was sealed by circumstances beyond its control: it outgrew the Newton, was eventually spun out into its own project and, as Apple gradually ran out of money and started laying people off, killed after about 3 years and never really saw industrial use. Subsequently a new crop of languages emerged on the coattails of the web, and Dylan, multimethods, and pretty much all interesting 80s and early-90s language technology vanished for over a decade.

![](http://venge.net/graydon/bottleneck.png)

## Perl, PHP, Java, Javascript, the web, and performance

Language technology goes through [population bottlenecks](http://en.wikipedia.org/wiki/Population_bottleneck). This is something any honest student of language history has to appreciate: events impose themselves. Old niches dry up, new niches open up, sometimes suddenly. A whole pile of good technology might get lost in such a bottleneck, like perfectly pleasant land creatures swept out to sea by a storm, suddenly selected-against for their ability to swim to a distant shore.

![](http://venge.net/graydon/perl.jpg)![](http://venge.net/graydon/java.jpeg)

The transitions from mainframes to minicomputers, and from minis to micros, each represented such bottlenecks. To an extent, the AI winters were bottlenecks.

The very sudden flourishing of the web was such a bottleneck. A ***major*** bottleneck. Suddenly the only thing that mattered was how easy it was to interoperate with the web, how "web-centric" your language managed to be, even if it was just a matter of perception.

![](http://venge.net/graydon/tiobestat.png)

I will not delve too far into the individual cases here -- how Java came to be seen as web-centric, the anguished competition about mobile code, the humble origins of PHP and Perl, the [Sun-Netscape alliance](http://en.wikipedia.org/wiki/IPlanet) and the amusing trademark of Javascript -- but if you look at the before- and after-web snapshots of which languages dominate, a massive shift happened around the mid-late 1990s. Languages that would previously have been confined to the margins of computing suddenly had huge, active job markets. Vibrant online cultures of new developers, [library ecosystems](http://cpan.org), conferences, professional associations, the whole deal. Java took the #1 PL spot, and JS, PHP, Python, Ruby and Perl regularly took spots in the top 10.

![](http://venge.net/graydon/mark_sweep_compact.png)The web language bottleneck damaged interest in languages that weren't sufficiently "webby" but in the new environment, among the languages that made it through, something surprising happened: performance considerations of language features ... loosened. A **lot**. Techniques that used to be considered "too expensive to use in production" -- say, ubiquitous [virtual dispatch](http://en.wikipedia.org/wiki/IPlanet), [garbage collection](http://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29), [latent types](http://en.wikipedia.org/wiki/Latent_typing) \-\- became "acceptable costs", culturally. Partly because of another few turns of the crank of [Moore's law](http://en.wikipedia.org/wiki/Moore%27s_law), partly because green-field programmer productivity became a stronger advantage when starting young businesses, but I think mostly because the programs people were writing were dominated by network and database I/O, such that the CPU and memory costs were lost in the noise.

In fact, the first couple generations of "web languages" were *abysmally inefficient*. [Indirect-threaded](http://en.wikipedia.org/wiki/Threaded_code) bytecode interpreters were the *fast* case: many were just AST-walking interpreters. PHP initially implemented its loops by `fseek()` in the source code. It's a testament to the amount of effort that had to go into building the *other parts* of the web -- the protocols, security, naming, linking and information-organization aspects -- that the programming languages underlying it all could be pretty much *anything*, technology-wise, so long as they were sufficiently web-friendly.

![](http://venge.net/graydon/inlinecaches.png)Of course, performance always *eventually* returns to consideration -- computers are about speed, fundamentally -- and the more-dynamic nature of many of the web languages eventually meant (re)deployment of many of the old performance-enhancing tricks of the Lisp and Smalltalk family, in order to speed up later generations of the web languages: generational GC, JITs, [runtime type analysis and specialization](http://en.wikipedia.org/wiki/Adaptive_optimization), and [polymorphic inline caching](http://en.wikipedia.org/wiki/Inline_caching) in particular. None of these were "new" techniques; but it was new for industry to be willing to rely *completely* on languages that benefit, or even require, such techniques in the first place.

In a sense, 20 years after the web smashed the language landscape to pieces, we've come to a place where the criteria for a "fast language" has been nudged forwards over the line that was holding back a lot of techniques that were cutting edge for the Lisp family at the beginning of the second AI winter. So long as you use "sufficiently JIT-friendly" dynamic language technology, you're good.

![](http://venge.net/graydon/julia.png)

## Julia

At last we come to [Julia](http://julialang.org/). A new language. A *great* language. A language I want my readers to be excited about, not just because it has some snazzy benchmark numbers, but also because of its history. Because of what it *means* in the broader sense of language technology.

Julia's a descendant of Dylan. A really, really good one.

Julia has all the tidy signs of Lisps: it's easily syntactically extensible, highly interactive, reflective, open, but also built around a solid, minimal, very clearly defined semantics that admits efficient compilation. This latter part is important: many people are drawn to Julia for its speed, and that speed is primarily a result of a very well thought out implementation of multimethods. These are the core design abstraction in the language, and Julia's done great work here. I encourage anyone who's bothered to read this far to read [Karpinski's recent presentation](http://nbviewer.ipython.org/gist/StefanKarpinski/b8fe9dbb36c1427b9f22) or, if you have the time, the full original [Julia paper](http://arxiv.org/abs/1209.5145) from 2012. It's an absolute delight.

![](http://venge.net/graydon/mit-lambda.png)

It is fast, it's true. It's not "C++ fast" but the space of people who demand languages be "C++ fast" is comparatively smaller, these days, than it was 20 years ago. It also trades space for speed, in a way that many JIT+GC languages do: lots of memory pressure from the code cache and uncollected garbage. Those caveats aside: it's fast enough to outpace most of its contemporaries in the REPL-centric space of scientific computing, at least partly because most of that space is dominated by Ousterhout-style hybrids.

Julia, like Dylan and Lisp before it, is a Goldilocks language. Done by a bunch of Lisp hackers who seriously know what they're doing.

It is trying to span the entire spectrum of its target users' needs, from numerical inner loops to glue-language scripting to dynamic code generation and reflection. And it's doing a very credible job at it. Its designers have produced a language that seems to be a strict improvement on Dylan, which was itself excellent. Julia's multimethods are type-parametric. It ships with really good multi-language FFIs, green coroutines and integrated package management. Its codegen is LLVM-MCJIT, which is as good as it gets these days.

To my eyes, Julia is one of the brightest spots in the recent language-design landscape; it's working in a space that really needs good languages right now; and it's reviving a language-lineage that could really do with a renaissance. I'm excited for its future.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=189377) comments
