---
title: Finding near-duplicates with Jaccard similarity and MinHash
url: https://blog.nelhage.com/post/fuzzy-dedup/
published: "2024-07-03T23:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/fuzzy-dedup/
---

# Finding near-duplicates with Jaccard similarity and MinHash

Suppose we have a large collection of documents, and we wish you identify which documents are approximately the same as each other. For instance, we may have crawled the web over some period of time, and expect to have fetched the “same page” several times, but to see slight differences in metadata, or that we have several revisions of a page following small edits. In this post I want to explore the method of approximate deduplication via Jaccard similarity and the MinHash approximation trick.
