---
title: '[$] Reconsidering x32 — again'
url: https://lwn.net/Articles/1074897/
published: "2026-06-01T14:22:19Z"
feed: lwn
guid: https://lwn.net/Articles/1074897/
---

# [$] Reconsidering x32 — again

The [x32 ABI](https://en.wikipedia.org/wiki/X32_ABI) was meant to be the best of both worlds, providing the expanded registers and instruction set of the x86-64 architecture while preserving the lower memory use of 32-bit systems. The Linux kernel has supported x32 since the 3.4 release in 2012. The initial excitement around x32 did not last, though, and kernel developers are considering removing that support — and not for the first time. Even the most unloved features tend to have a few users, though, making removal hard.
