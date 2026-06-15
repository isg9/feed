---
title: sir talks-a-lot — wingolog
url: https://wingolog.org/archives/2023/12/12/sir-talks-a-lot
published: "2023-12-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/12/12/sir-talks-a-lot
---

# sir talks-a-lot — wingolog

## [sir talks-a-lot](/archives/2023/12/12/sir-talks-a-lot)

12 December 2023 3:18 PM

- [meta](/tags/meta)
- [talks](/tags/talks)
- [presentations](/tags/presentations)
- [guile](/tags/guile)
- [webassembly](/tags/webassembly)
- [igalia](/tags/igalia)
- [icymi](/tags/icymi)

I know, dear reader: of course you have already seen all my talks this year. Your attentions are really too kind and I thank you. But those other people, maybe you share one talk with them, and then they ask you for more, and you have to go stalking back through the archives to slake their nerd-thirst. This happens all the time, right?

I was thinking of you this morning and I said to myself, why don’t I put together a post linking to all of my talks in 2023, so that you can just send them a link; here we are. You are very welcome, it is really my pleasure.

### 2023 talks

*Scheme + Wasm + GC = MVP: Hoot Scheme-to-Wasm compiler update.* Wasm standards group, Munich, 11 Oct 2023. [slides](https://wingolog.org/pub/wasm-cg-2023-10-scheme-slides.pdf)

*Scheme to Wasm: Use and misuse of the GC proposal.* Wasm GC subgroup, 18 Apr 2023. [slides](https://wingolog.org/pub/wasm-gc-2023-04-scheme-slides.pdf)

*A world to win: WebAssembly for the rest of us.* BOB, Berlin, 17 Mar 2023. [blog](https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us) [slides](https://wingolog.org/pub/2023-bobkonf-wasm-for-the-rest-of-us-slides.pdf) [youtube](https://www.youtube.com/watch?v=lUCegCa7A08&pp=ygUKYW5keSB3aW5nbw%3D%3D)

*Cross-platform mobile UI: “Compilers, compilers everywhere”.* EOSS, Prague, 27 June 2023. [slides](https://wingolog.org/pub/2023-eoss-cross-platform-ui-slides.pdf) [youtube](https://www.youtube.com/watch?v=r6ctxZphtkI&pp=ygUKYW5keSB3aW5nbw%3D%3D) [blog](https://wingolog.org/archives/2023/06/15/parallel-futures-in-mobile-application-development) [blog](https://wingolog.org/archives/2023/05/02/structure-and-interpretation-of-ark) [blog](https://wingolog.org/archives/2023/04/26/structure-and-interpretation-of-flutter) [blog](https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript) [blog](https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native) [blog](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)

*CPS Soup: A functional intermediate language.* Spritely, remote, 10 May 2023. [blog](https://wingolog.org/archives/2023/05/20/approaching-cps-soup) [slides](https://wingolog.org/pub/2023-spritely-cps-soup-slides.pdf)

*Whippet: A new GC for Guile.* FOSDEM, Brussels, 4 Feb 2023. [blog](https://wingolog.org/archives/2023/02/07/whippet-towards-a-new-local-maximum) [event](https://fosdem.org/2023/schedule/event/whippet/) [slides](https://wingolog.org/pub/fosdem-whippet-2023-slides.pdf)

### but wait, there’s more

Still here? The full [talks archive](//wingolog.org/talks/) will surely fill your cup.

## related articles

- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)

### One response

1. Hubert says:[13 December 2023 2:39 PM](#ee8ec16cccbf28064566d505c5d2f31349309f19)

   Thanks! That should keep me busy for a while (eh-eh-eh).

   I’ve had a burning question about guile for some time now. You’ve managed to speed it up by (approx.) 15x between versions 1.7 and 3.0, for tak and mazefun benchmarks, which is outstanding. Guile’s performance in now between that of interpreted Petite Chez and compiled Gambit Scheme on those codes (last I checked, couple years back). The continuation-based ctak on the other hand does not seem to have seen a similar performance uplift. I find this puzzling because guile is now a produce of (yummy) CPS soup, from which I (naively?) would assume that continuation-based code would enjoy a most tasty spike in performance. Is enhancing call/cc perf on the TOD list of guile development, or is there something more fundamental (a design decision? something to do with fibers? or FFI?) that might be at play in this?

Comments are closed.
