---
title: the wax wing bits — wingolog
url: https://wingolog.org/archives/2007/11/14/the-wax-wing-bits
published: "2007-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/11/14/the-wax-wing-bits
---

# the wax wing bits — wingolog

## [the wax wing bits](/archives/2007/11/14/the-wax-wing-bits)

15 November 2007 4:32 AM

- [scheme](/tags/scheme)
- [r6rs](/tags/r6rs)
- [ikarus](/tags/ikarus)
- [death from above](/tags/death%20from%20above)

Inspired by the birth of Abdulaziz Ghuloum's baby, [Ikarus Scheme](http://www.cs.indiana.edu/~aghuloum/ikarus/), allow me to spend electrons on the latest Scheme standard, [R6RS](http://r6rs.org/).

I can imagine three camps into which you might fall. Members of the first have read Ghuloum's tantalizing 2006 paper, [An Incremental Approach to Compiler Construction](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf), and since have been wondering what this guy was up to. The second camp needs to read the aforementioned paper. Folks in the third group don't give a shit about hacking, but that's cool, diffrent strokes diffrent fokes and such.

So in answer to the first group, Ghuloum recently released his incremental, optimizing compiler for R6RS Scheme. Among other reasons, Ikarus promises to be important for its lucid synopsis of the new Scheme standard in its [User's Guide](http://www.cs.indiana.edu/~aghuloum/ikarus/ikarus-users-guide.pdf). I quote, verbosely:

> The major difference between R5RS \[the last scheme revision\] and R6RS is the way in which programs are loaded and evaluated.
>
> In R5RS, Scheme implementations typically start as an interactive session (often referred to as the REPL, or read-eval-print-loop). Inside the interactive session, the user enters definitions and expressions one at a time using the keyboard. Files, which also contain definitions and expressions, can be loaded and reloaded by calling the load procedure. The environment in which the interactive session starts often contains implementation-specific bindings that are not found R5RS and users may redefine any of the initial bindings. The semantics of a loading a file depends on the state of the environment at the time the file contents are evaluated.
>
> R6RS differs from R5RS in that it specifies how whole programs, or scripts, are compiled and evaluated. An R6RS script is closed in the sense that all the identifiers found in the body of the script must either be defined in the script or imported from a library. R6RS also specifies how libraries can be defined and used. While files in R5RS are loaded imperatively into the top-level environments, R6RS libraries can be imported declaratively in scripts and in other R6RS libraries.

I got through most of the standard on plane rides last month, and had been meaning to write about it, but he does so better than I would have. The upshot is that R6RS is a kind of "static scheme", not really the [minsky-style](http://lambda-the-ultimate.org/node/12) programming that has been the hallmark of Lisp.

I don't know how I feel about this. On the one hand, the editors appear to have done an excellent job specifying a system of libraries that will allow incremental, separate compilation, that interacts well with macros, and that even allows libraries to be loaded into read-only memory, shared across processes. On the other, they completely neglected the dynamic roots of the language.

In the R6RS rationale document, the editors choose to [spin the lack of specification the other way](http://www.r6rs.org/final/html/r6rs-rationale/r6rs-rationale-Z-H-10.html#node_chap_8):

> Tightening the specification of programs from R5RS would have been possible, but could have restricted the design employed by Scheme implementations in undesirable ways. Moreover, alternative approaches to structuring the user interface of a Scheme implementation have emerged since R5RS. Consequently, R6RS makes no attempt at trying to specify the semantics of programs as in R5RS; the design of an interactive environment is now completely in the hands of the implementors.

The reaction of the internet to the R6RS process has been mixed, but largely negative. I believe that this is mostly because of dissenters being more vocal than proponents, who had the inertia of process on their side; that, and pissants that write more than they hack.

Still, when the picture is presented as clearly as Ghuloum does, the R6RS proposal has an enormous gaping hole, viz. how to interactively develop programs. No long-running web servers, interactively added to over the course of a year; nothing that would make [SLIME](http://common-lisp.net/project/slime/) for Scheme possible; even the REPL itself is undefined in the new standard.

What happens now will largely be a social phenomenon, I think. Those that develop "statically" are probably happy with R6. Those that develop more dynamically either fork, switch languages, or find a way to reintroduce dynamism into the subset of the community that produced R6. Those in the middle... well, you're in good company, I guess.

**tunes**

Depends on your mood, but Death From Above 1979 has fit mine in recent days. "Going Steady" [rocks out](http://www.threadless.com/product/456/Rock_Out_With_Your_Cock_Out).

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [\-\\*\- text -\*-](/archives/2007/05/22/text)
- [exciting papers!](/archives/2006/09/18/exciting-papers)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)

### 2 responses

1. [Nathan Myers](http://advogato.org/person/ncm/) says:[15 November 2007 8:01 AM](#2764)

   It sounds like the R6RS people have done something astonishingly sensible and mature, especially considering their constituency. My prediction is that the people who want to write useful programs in Scheme will embrace R6RS joyfully, and the rest will continue doing whatever the hell they've been doing. That seems like real progress.

2. David Holmes says:[16 November 2007 4:10 AM](#2797)

   I just want to thank you for bringing Ghuloum's paper and project to my attention. As somebody who attempted to study compilers in the past but never completed even a working "toy" compiler, this paper fascinates me and has renewed my interest in the subject.

Comments are closed.
