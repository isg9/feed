---
title: Reader/reader blocking in reader/writer locks
url: https://blog.nelhage.com/post/rwlock-contention/
published: "2019-05-07T15:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/rwlock-contention/
---

# Reader/reader blocking in reader/writer locks

Abstract In writer-priority reader/writer locks, as soon as a single writer enters the acquisition queue, all future accesses block behind any in-flight reads. Thus, if any readers hold the lock for extended periods of time, this can lead to extreme pauses and loss of throughput given even a very small number of writers. This phenomenon is well-known in certain systems engineering communities (e.g. among some kernel or database developers), but is often surprising when first encountered, and has important implications for the design of such systems.
