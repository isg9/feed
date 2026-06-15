---
title: '[$] An update on fanotify'
url: https://lwn.net/Articles/1075829/
published: "2026-06-08T15:35:10Z"
feed: lwn
guid: https://lwn.net/Articles/1075829/
---

# [$] An update on fanotify

In a filesystem-track session at the 2026 [Linux Storage,
Filesystem, Memory Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/), Amir Goldstein updated attendees on the [fanotify](https://man7.org/linux/man-pages/man7/fanotify.7.html) filesystem-event monitoring subsystem. He wanted to describe changes that had come in the last year or so, as well as upcoming features and some remaining challenges in his efforts [to use fanotify for hierarchical
storage management](https://lwn.net/Articles/981392/) (HSM). Fanotify is the user-space API for monitoring files, directories, and filesystems for events of various sorts (e.g. opening or deleting a file).
