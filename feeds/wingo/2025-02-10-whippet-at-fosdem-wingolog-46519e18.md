---
title: whippet at fosdem — wingolog
url: https://wingolog.org/archives/2025/02/10/whippet-at-fosdem
published: "2025-02-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/02/10/whippet-at-fosdem
---

# whippet at fosdem — wingolog

## [whippet at fosdem](/archives/2025/02/10/whippet-at-fosdem)

10 February 2025 9:01 PM

- [whippet](/tags/whippet)
- [fosdem](/tags/fosdem)
- [garbage collection](/tags/garbage%20collection)
- [whiffle](/tags/whiffle)
- [guile](/tags/guile)
- [igalia](/tags/igalia)

Hey all, the video of my [FOSDEM talk on Whippet](https://fosdem.org/2025/schedule/event/fosdem-2025-6066-the-whippet-embeddable-garbage-collection-library/) is up:

Slides [here](https://wingolog.org/pub/fosdem-whippet-2025-slides.pdf), if that’s your thing.

I ended the talk with some puzzling results around generational collection, which prompted [yesterday’s
post](https://wingolog.org/archives/2025/02/09/baffled-by-generational-garbage-collection). I don’t have a firm answer yet. Or rather, perhaps for the splay benchmark, it is to be expected that a generational GC is not great; but there are other benchmarks that also show suboptimal throughput in generational configurations. Surely it is some tuning issue; I’ll be looking into it.

Happy hacking!

## related articles

- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)

### 2 responses

1. Ty says:[11 February 2025 3:14 AM](#a33a7bd541fefc69216d0155cbb984e12980803d)

   Is there a mirror of the video? This one pauses to buffer a few times per second for me

2. [wingo](https://wingolog.org) says:[11 February 2025 8:39 AM](#9f91ac4811bfe4e9d641991981a93717d03e47ca)

   Original video links here: [https://fosdem.org/2025/schedule/event/fosdem-2025-6066-the-whippet-embeddable-garbage-collection-library/](https://fosdem.org/2025/schedule/event/fosdem-2025-6066-the-whippet-embeddable-garbage-collection-library/)

   They probably should have provided lower bitrate transcodings :)

Comments are closed.
