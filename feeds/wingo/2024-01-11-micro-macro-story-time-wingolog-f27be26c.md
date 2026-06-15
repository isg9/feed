---
title: micro macro story time — wingolog
url: https://wingolog.org/archives/2024/01/11/micro-macro-story-time
published: "2024-01-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/01/11/micro-macro-story-time
---

# micro macro story time — wingolog

## [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

11 January 2024 2:10 PM

- [scheme](/tags/scheme)
- [macros](/tags/macros)
- [syntax expansion](/tags/syntax%20expansion)
- [psyntax](/tags/psyntax)
- [dybvig](/tags/dybvig)
- [guile](/tags/guile)
- [igalia](/tags/igalia)

Today, a tiny tale: [about 15 years
ago](https://wingolog.org/archives/2009/03/12/on-psyntax) I was working on [Guile’s macro
expander](https://git.savannah.gnu.org/cgit/guile.git/tree/module/ice-9/psyntax.scm). Guile inherited this code from an early version of Kent Dybvig’s [portable syntax
expander](https://conservatory.scheme.org/psyntax/r6rs-libraries/). It was... not easy to work with.

Some difficulties were essential. Scope is tricky, after all.

Some difficulties were incidental, but deep. The expander is ultimately a function that translates Scheme-with-macros to Scheme-without-macros. However, it is itself written in Scheme-with-macros, so to load it on a substrate without macros requires a [pre-expanded copy of
itself](https://git.savannah.gnu.org/cgit/guile.git/tree/module/ice-9/psyntax-pp.scm), whose data representations need to be compatible with any incremental change, so that you will be able to use the new expander to produce a fresh pre-expansion. This difficulty could have been avoided by [incrementally bootstrapping the
library](https://github.com/schierlm/guile-psyntax-bootstrapping). It works once you are used to it, but it’s gnarly.

But then, some difficulties were just superflously egregious. Dybvig is a totemic developer and researcher, but a generation or two removed from me, and when I was younger, it never occurred to me to just email him to ask why things were this way. (A tip to the reader: if someone is doing work you are interested in, you can just email them. Probably they write you back! If they don’t respond, it’s not you, they’re probably just busy and their inbox leaks.) Anyway in my totally speculatory reconstruction of events, when Dybvig goes to submit his algorithm for publication, he gets annoyed that “expand” doesn’t sound fancy enough. In a way it’s similar to the original SSA developers [thinking that “phony functions” wouldn’t get
published](https://wingolog.org/archives/2023/05/20/approaching-cps-soup).

So Dybvig calls the expansion function “χ”, because the Greek chi looks like the X in “expand”. Fine for the paper, whatever paper that might be, but then in `psyntax`, there are all these functions named `chi` and `chi-lambda` and all sorts of nonsense.

In early years I was often confused by these names; I wasn’t in on the pun, and I didn’t feel like I had enough responsibility for this code to think what the name should be. I finally broke down and changed all instances of “chi” to “expand” back in 2011, and never looked back.

Anyway, this is a story with a very specific moral: don’t name your functions `chi`.

## related articles

- [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)

### 3 responses

1. [Jean Abou Samra](https://jean.abou-samra.fr) says:[11 January 2024 6:42 PM](#01d2a775e733e354b8992c16d9cabf2642f0b523)

   Anyway in my totally speculatory reconstruction of events, when Dybvig goes to submit his algorithm for publication, he gets annoyed that “expand” doesn’t sound fancy enough. In a way it’s similar to the original SSA developers thinking that “phony functions” wouldn’t get published.

   I think the reason might be much simpler. Virtually all research papers are written in TeX, and it’s annoying to type mathematical symbols with multi-letter names in TeX: `\mathrm{expand}` (NB: I had to look up the command) vs. just `\xhi`. Don’t underestimate the impact of something that simple… (It’s in a way similar to why most code is written in ASCII.)

2. Hubert says:[12 January 2024 3:34 AM](#80ac148aa956b500fc972d6bd45760610bab12b8)

   I have email from 5 years ago that I have yet to reply to. It is not always about “being busy”, many times it is just that this particular email was most interesting and sparked a thought trail, requiring (or evoking the need for) further thought to fully address (and thinking can easily extend over decades, and more)!

3. [Delon Newman](https://delonnewman.name) says:[13 January 2024 10:01 PM](#8aa274ca55d42ae7f373f7f22f456f206717a80f)

   Great article. I love the tip to reach out to people who are doing work you’re interested in. You may hear from me!

Comments are closed.
