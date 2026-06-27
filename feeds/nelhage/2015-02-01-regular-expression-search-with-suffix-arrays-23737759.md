---
title: Regular Expression Search with Suffix Arrays
url: https://blog.nelhage.com/2015/02/regular-expression-search-with-suffix-arrays/
published: "2015-02-01T15:52:43Z"
feed: nelhage
guid: https://blog.nelhage.com/2015/02/regular-expression-search-with-suffix-arrays/
---

# Regular Expression Search with Suffix Arrays

Back in January of 2012, Russ Cox posted an excellent blog post detailing how Google Code Search had worked, using a trigram index. By that point, I’d already implemented early versions of my own livegrep source-code search engine, using a different indexing approach that I developed independently, with input from a few friends. This post is my long-overdue writeup of how it works. Suffix Arrays A suffix array is a data structure used for full-text search and other applications, primarily these days in the field of bioinformatics.
