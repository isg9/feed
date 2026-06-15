---
title: Making the Sentence Structure of Paragraphs Apparent
url: https://blog.regehr.org/archives/618
published: "2011-10-25T15:42:38Z"
feed: regehr
guid: http://blog.regehr.org/?p=618
---

# Making the Sentence Structure of Paragraphs Apparent

This post is about a tiny thing that makes a big difference in practice because I spend so much time writing. Usually, people compose paragraphs as monolithic blocks of text. For several years now, I’ve written paragraphs like this:

> ```
> Integer overflow bugs in C and C++ programs are difficult to track
> down and may lead to fatal errors or exploitable vulnerabilities.
> %
> Although a number of tools for finding these bugs exist, the
> situation is complicated because not all overflows are bugs.
> %
> Better tools need to be constructed---but a thorough understanding
> of the issues behind these errors does not yet exist.
> ```

For the non-LaTeX users out there, the percent symbol indicates a comment line. When this text is typeset, the sentences will flow together in the usual fashion. Why do I do it this way? The most important reason is that it calls my attention to the individual sentences in a paragraph. Frequent offenders like run-on sentences, paragraphs that lack a topic sentence, groups of sentences with repetitive structure, and paragraphs that contain a single sentence become trivial to spot. I often find that when I take normally formatted text — whether written by me or by a co-author — and split it into sentences like this, hidden problems in the writing become obvious. A secondary benefit comes out only when interacting with a revision control system: because editing individual sentences does not cause an entire paragraph to require re-wrapping, diffs are much easier to read and conflicts become less likely. The cost of writing this way is that I suspect it annoys co-authors sometimes.

There must be other tricks like this that people use — I’d be interested to learn about them. As a random example, when using some old word processor (MultiMate, I think — but modern word processors support this as well) I used to use a switch that made certain non-printing characters visible.
