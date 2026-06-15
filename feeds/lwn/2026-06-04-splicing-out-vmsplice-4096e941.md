---
title: '[$] Splicing out vmsplice()'
url: https://lwn.net/Articles/1075838/
published: "2026-06-04T16:22:46Z"
feed: lwn
guid: https://lwn.net/Articles/1075838/
---

# [$] Splicing out vmsplice()

The [`splice()`](https://man7.org/linux/man-pages/man2/splice.2.html) and [`vmsplice()`](https://man7.org/linux/man-pages/man2/vmsplice.2.html) system calls are meant to improve performance for certain data-movement tasks by minimizing (or avoiding altogether) system calls and the copying of data. They also have a long history of security problems. The recent flood of LLM-discovered vulnerabilities has drawn attention, once again, to `splice()` and `vmsplice()`; as a result, they may end up being removed altogether.
