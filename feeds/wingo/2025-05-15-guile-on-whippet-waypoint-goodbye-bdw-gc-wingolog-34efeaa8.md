---
title: 'guile on whippet waypoint: goodbye, bdw-gc? — wingolog'
url: https://wingolog.org/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc
published: "2025-05-15T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc
---

# guile on whippet waypoint: goodbye, bdw-gc? — wingolog

## [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)

15 May 2025 2:39 PM

- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [bdw](/tags/bdw)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [weak references](/tags/weak%20references)
- [ephemerons](/tags/ephemerons)
- [nofl](/tags/nofl)

Hey all, just a lab notebook entry today. I’ve been working on the [Whippet GC library](https://github.com/wingo/whippet) for about three years now, learning a lot on the way. The goal has always been to replace [Guile’s use of the Boehm-Demers-Weiser
collector](https://wingolog.org/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs) with something more modern and maintainable. Last year I finally got to the point that I felt [Whippet was
feature-complete](https://wingolog.org/archives/2024/09/18/whippet-progress-update-feature-complete), and taking into account the old adage about long arses and brief videos, I think that wasn’t too far off. I carved out some time this spring and for the last month have been integrating Whippet into Guile in anger, on the [wip-whippet
branch](https://git.savannah.gnu.org/cgit/guile.git/log/?h=wip-whippet).

### the haps

Well, today I removed the last direct usage of the BDW collector’s API by Guile! Instead, Guile uses Whippet’s API any time it needs to allocate an object, add or remove a thread from the active set, identify the set of roots for a collection, and so on. Most tracing is still conservative, but this will move to be more precise over time. I haven’t had the temerity to actually try one of the Nofl-based collectors yet, but that will come soon.

Code-wise, the initial import of Whippet added some 18K lines to Guile’s repository, as counted by `git diff --stat`, which includes documentation and other files. There was an unspeakable amount of autotomfoolery to get Whippet in Guile’s ancient build system. Changes to Whippet during the course of integration added another 500 lines or so. Integration of Whippet removed around 3K lines of C from Guile. It’s not a pure experiment, as my branch is also a major version bump and so has the freedom to refactor and simplify some things.

Things are better but not perfect. Notably, I switched to build weak hash tables in terms of buckets and chains where the links are ephemerons, which give me concurrent lock-free reads and writes but not resizable tables. I would like to somehow resize these tables in response to GC, but haven’t wired it up yet.

Anyway, next waypoint will be trying out the version of Whippet’s [Nofl-based mostly-marking collector that traces all heap edges
conservatively](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md#conservative-heap-scanning). If that works... well if that works... I don’t dare to hope! We will see what we get when that happens. Until then, happy hacking!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [whippet hacklog: adding freelists to the no-freelist space](/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)

Comments are closed.
