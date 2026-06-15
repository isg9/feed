---
title: a whippet waypoint — wingolog
url: https://wingolog.org/archives/2025/05/09/a-whippet-waypoint
published: "2025-05-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/05/09/a-whippet-waypoint
---

# a whippet waypoint — wingolog

## [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)

9 May 2025 9:36 PM

- [whippet](/tags/whippet)
- [academia](/tags/academia)
- [papers](/tags/papers)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [guile](/tags/guile)
- [igalia](/tags/igalia)

Hey peoples! Tonight, some meta-words. As you know I am fascinated by compilers and language implementations, and I just want to know all the things and implement all the fun stuff: intermediate representations, flow-sensitive source-to-source optimization passes, register allocation, instruction selection, garbage collection, all of that.

It started long ago with a combination of curiosity and a hubris to satisfy that curiosity. The usual way to slake such a thirst is structured higher education followed by industry apprenticeship, but for whatever reason my path sent me through a nuclear engineering bachelor’s program instead of computer science, and continuing that path was so distasteful that I noped out all the way to rural Namibia for a couple years.

Fast-forward, after 20 years in the programming industry, and having picked up some language implementation experience, a few years ago I returned to garbage collection. I have a good level of language implementation chops but never wrote a memory manager, and Guile’s performance was limited by its use of the Boehm collector. I had been on the lookout for something that could help, and when I learned of [Immix](https://dl.acm.org/doi/10.1145/1379022.1375586) it seemed to me that the only thing missing was an appropriate implementation for Guile, and hey I could do that!

### whippet

I started with the idea of an [MMTk](https://www.mmtk.io/)-style interface to a memory manager that was abstract enough to be implemented by a variety of different collection algorithms. This kind of abstraction is important, because in this domain it’s easy to convince oneself that a given algorithm is amazing, just based on vibes; to stay grounded, I find I always need to compare what I am doing to some fixed point of reference. This GC implementation effort grew into [Whippet](https://github.com/wingo/whippet), but as it did so a funny thing happened: the [mark-sweep collector that I
prototyped](https://github.com/wingo/whippet/commit/7b85284a89ccdd1fe34754c3f8c18903254d6f1f) as a direct replacement for the Boehm collector maintained mark bits in a side table, which I realized was a suitable substrate for Immix-inspired bump-pointer allocation into holes. I ended up building on that to develop an Immix collector, but without lines: instead each granule of allocation (16 bytes for a 64-bit system) is its own line.

### regions?

The [Immix paper](https://dl.acm.org/doi/10.1145/1379022.1375586) is funny, because it defines itself as a new class of *mark-region* collector, fundamentally different from the three other fundamental algorithms (mark-sweep, mark-compact, and evacuation). Immix’s *regions* are blocks (64kB coarse-grained heap divisions) and lines (128B “fine-grained” divisions); the innovation (for me) is the *optimistic evacuation* discipline by which one can potentially defragment a block without a second pass over the heap, while also allowing for bump-pointer allocation. See the papers for the deets!

However what, really, are the regions referred to by *mark-region*? If they are blocks, then the concept is trivial: everyone has a block-structured heap these days. If they are spans of lines, well, how does one choose a line size? As I understand it, Immix’s choice of 128 bytes was to be fine-grained enough to not lose too much space to fragmentation, while also being coarse enough to be eagerly swept during the GC pause.

This constraint was odd, to me; all of the mark-sweep systems I have ever dealt with have had lazy or concurrent sweeping, so the lower bound on the line size to me had little meaning. Indeed, as one reads papers in this domain, it is hard to know the real from the rhetorical; the review process prizes novelty over nuance. Anyway. What if we cranked the precision dial to 16 instead, and had a line per granule?

That was the process that led me to Nofl. It is a space in a collector that came from mark-sweep with a side table, but instead uses the side table for bump-pointer allocation. Or you could see it as an Immix whose line size is 16 bytes; it’s certainly easier to explain it that way, and that’s the tack I took in a [recent paper submission to
ISMM’25](https://arxiv.org/abs/2503.16971).

### paper??!?

Wait what! I have a fine job in industry and a blog, why write a paper? Gosh I have meditated on this for a long time and the answers are very silly. Firstly, one of my language communities is Scheme, which was a research hotbed some 20-25 years ago, which means many practitioners—people I would be pleased to call peers—came up through the PhD factories and published many interesting results in academic venues. These are the folks I like to hang out with! This is also what academic conferences are, chances to shoot the shit with far-flung fellows. In Scheme this is fine, my work on Guile is enough to pay the intellectual cover charge, but I need more, and in the field of GC I am not a proven player. So I did an atypical thing, which is to cosplay at being an independent researcher without having first been a dependent researcher, and just solo-submit a paper. Kids: if you see yourself here, just go get a doctorate. It is not easy but I can only think it is a much more direct path to goal.

And the result? Well, friends, it is this blog post :) I got the usual assortment of review feedback, from the very sympathetic to the less so, but ultimately people were confused by leading with a comparison to Immix but ending without an evaluation against Immix. This is fair and the paper does not mention that, you know, I don’t have an Immix lying around. To my eyes it was a good paper, an [80%
paper](https://mastodon.social/@gwenthekween@kitsunes.club/114461573838668403), but, you know, just a try. I’ll try again sometime.

In the meantime, I am driving towards getting Whippet into Guile. I am hoping that sometime next week I will have excised all the uses of the BDW (Boehm GC) API in Guile, which will finally allow for testing Nofl in more than a laboratory environment. Onwards and upwards!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)

Comments are closed.
