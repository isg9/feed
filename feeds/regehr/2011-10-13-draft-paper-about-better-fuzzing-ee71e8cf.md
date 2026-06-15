---
title: Draft Paper about Better Fuzzing
url: https://blog.regehr.org/archives/595
published: "2011-10-13T17:24:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=595
---

# Draft Paper about Better Fuzzing

![](https://blog.regehr.org/wp-content/uploads/2011/10/boss_fz_5_fuzz-600-600-e1318526543265.jpg) The other day I [posted](https://blog.regehr.org/archives/591) about a simple, low-effort way to improve the bug-finding performance of a random tester. We now have a [draft paper about this topic](http://www.cs.utah.edu/~regehr/papers/swarm12.pdf), it’s joint work between my group at Utah and Alex Groce’s group at Oregon State. The key claim is:

> … for realistic systems, randomly excluding some features from some tests can improve coverage and fault detection, compared to a test suite that potentially uses every feature in every test. The benefit of using of a single inclusive default configuration– that every test can potentially expose any fault and cover any behavior, heretofore usually taken for granted in random testing–does not, in practice, make up for the fact that some features can, statistically, suppress behaviors.

We’d be interested in feedback.
