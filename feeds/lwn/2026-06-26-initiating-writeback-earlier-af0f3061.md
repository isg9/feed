---
title: '[$] Initiating writeback earlier'
url: https://lwn.net/Articles/1078767/
published: "2026-06-26T17:14:30Z"
feed: lwn
guid: https://lwn.net/Articles/1078767/
---

# [$] Initiating writeback earlier

Writeback is the process of ensuring that dirty pages or folios in the page cache are flushed to the disk, so that changes to those files are made persistent. In a filesystem-track session at the 2026 [Linux Storage,
Filesystem, Memory Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/), Jeff Layton wanted to discuss whether the writeback operation should be initiated earlier than it is today. The consensus seemed to be that it should be done earlier, but the path toward making that happen was less clear.
