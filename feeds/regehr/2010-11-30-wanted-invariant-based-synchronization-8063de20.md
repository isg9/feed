---
title: 'Wanted: Invariant-Based Synchronization'
url: https://blog.regehr.org/archives/313
published: "2010-11-30T16:43:54Z"
feed: regehr
guid: http://blog.regehr.org/?p=313
---

# Wanted: Invariant-Based Synchronization

Although a significant fraction of the programming languages community works on detecting race conditions in multi-threaded software, I haven’t been able to get very excited about this. Certainly race-free programs have some nice properties, but race freedom is neither a necessary nor sufficient condition for concurrency correctness. This research area doesn’t feel to me like it is fundamental, but rather an evolutionary stage that has already worn out its welcome.

What I’d like to see — both as a developer and as a researcher — is a much more concrete link between concurrency control and program correctness. For example, in [invariant-based synchronization](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.8.8037&rep=rep1&type=pdf), we specify the invariants that must hold over concurrent data structures and then use partially automated methods to derive locking solutions that guarantee that the invariants are maintained. Coarse-grained solutions are easy to find, and of course fine-grained solutions require more work. The scarcity of papers on this kind of idea has surprised me for a while now. I predict that ideas similar to invariant-based synchronization will become a popular area of research when people get over races and transactions, probably in the 2015-2020 time frame.
