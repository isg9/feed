---
title: Future Directions for Optimizing Compilers
url: https://blog.regehr.org/archives/1619
published: "2018-09-10T04:44:28Z"
feed: regehr
guid: http://blog.regehr.org/?p=1619
---

# Future Directions for Optimizing Compilers

I wanted to write a manifesto-ish sort of piece about what compilers are supposed to look like in the future. [Nuno](http://web.ist.utl.pt/nuno.lopes/) was the obvious coauthor since I’ve talked to him about this topic so much that I’m overall not really sure which parts started out as his ideas and which were mine. The article didn’t end up feeling like a good fit for a series of blog posts, so we’ve placed it on arXiv: [Future Directions for Optimizing Compilers](https://arxiv.org/abs/1809.02161). The first paragraph of the paper gives the main idea:

> As software becomes larger, programming languages become higher-level, and processors continue to fail to be clocked faster, we’ll increasingly require compilers to reduce code bloat, eliminate abstraction penalties, and exploit interesting instruction sets. At the same time, compiler execution time must not increase too much and also compilers should never produce the wrong output. This paper examines the problem of making optimizing compilers faster, less buggy, and more capable of generating high-quality output.

We plan to keep revising the paper and would be happy to receive feedback on it.
