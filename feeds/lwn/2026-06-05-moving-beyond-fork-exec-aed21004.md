---
title: '[$] Moving beyond fork() + exec()'
url: https://lwn.net/Articles/1076018/
published: "2026-06-05T14:06:43Z"
feed: lwn
guid: https://lwn.net/Articles/1076018/
---

# [$] Moving beyond fork() + exec()

Since the earliest days of Unix, two of the core process-oriented system calls have been `fork()`, which creates a child process as a copy of the parent, and `exec()`, which runs a new program in the place of the current one. In Linux kernels, those system calls are better known as [`clone()`](https://man7.org/linux/man-pages/man2/clone.2.html) and [`execve()`](https://man7.org/linux/man-pages/man2/execve.2.html), but the core functionality remains the same. While there is elegance to this process-creation model, there are shortcomings as well. A recent [proposal](https://lwn.net/ml/all/20260528095235.2491226-1-me@linux.beauty) from Li Chen to add "spawn templates" to the kernel will not be accepted in its current form, but it may point the way toward a new process-creation primitive in the future.
