---
title: '[$] Representing the true signatures of kernel functions'
url: https://lwn.net/Articles/1073762/
published: "2026-06-01T18:59:43Z"
feed: lwn
guid: https://lwn.net/Articles/1073762/
---

# [$] Representing the true signatures of kernel functions

Optimizing compilers can, under some circumstances, infer when a parameter to a function is not needed, and remove it. This is all well and good until the kernel's tracing or BPF subsystems need information on how to call the function or where its arguments are stored. Alan Maguire and Yonghong Song spoke at the 2026 [Linux
Storage, Filesystem, Memory-Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/) about their work on recording information regarding changed function signatures in the kernel's BTF debugging information, to better support tracing such functions.
