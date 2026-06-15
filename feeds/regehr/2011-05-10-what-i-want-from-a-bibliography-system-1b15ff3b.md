---
title: What I Want From a Bibliography System
url: https://blog.regehr.org/archives/531
published: "2011-05-10T03:45:08Z"
feed: regehr
guid: http://blog.regehr.org/?p=531
---

# What I Want From a Bibliography System

As a professor I spend a fair amount of time wrangling with references. Because it’s free and reasonably simple, I use BibTeX: an add-on tool for LaTeX that automates the construction of a bibliography by pulling references out of a separate text file, assigning them numbers (or other identifiers), and formatting the entries appropriately.

BibTeX entries are widely available on the web. In principle this is great, but in practice the quality of downloadable BibTeX entries is poor: they contain so many typos, omissions, misclassifications, and other problems that I no longer download them, but rather create a new entry from scratch, perhaps using the existing entry as a rough specification. Even BibTeX entries from the ACM’s Digital Library — a for-pay service provided by the premier professional society for computer science — are not directly usable, basically every one of them requires editing. Over time I’ve come to more and more blame BibTeX for these problems, rather than blaming the people creating the entries. Here are the design points I’d like to see in a bibliography system.

**Style neutrality:** A bibliography entry should be a collection of factual information about a document. The “bibliography style” — a collection of formatting guidelines specific to a particular journal or similar — should be a separate concern. BibTeX does support bibliography styles but the implementation is incomplete, forcing entries to be tweaked when the style is changed.

**Minimal redundancy:** Any large BibTeX file contains a substantial amount of redundancy, because BibTeX poorly supports the kind of abstraction that would permit, for example, the common parts of two papers appearing at the same conference to be factored out. Duplication is bad because it consumes time and also makes it basically impossible to avoid inconsistencies.

**Standalone entries:** I should be able to download a bibliography entry from the net and use it right away. This goal is met by BibTeX but only at the expense of massive redundancy. Meeting both goals at the same time may be tricky. One solution would be to follow an entry’s dependency chain when exporting it, in order to rip a minimal, self-sufficient collection of information out of the database. This seems awkward. A better answer is probably to rely on conventions: if there is a globally recognized name for each conference and journal, a self-sufficient entry can simply refer to it.

**Plain-text entries:** Putting the bibliography in XML or some random binary format isn’t acceptable; the text-only database format is one of the good things about BibTeX.

**Network friendliness:** It should be easy to grab bibliography entries from well-known sources on the network, such as arXiv. Assuming that the problem of these entries sucking can be solved, I should not even need a local bibliography file at all.

**User friendliness:** BibTeX suffers from weak error checking (solved somewhat by add-on tools like bibclean) and formatting difficulties. For example, BibTeX’s propensity for taking capitalized letters from the bib entry and making them lowercase in the output causes numerous bibliographies I read to contain proper nouns that are not capitalized. Messing with BibTeX styles is quite painful.

BibTeX is a decent tool, it just hasn’t aged or scaled well. CrossTeX and BibLaTeX sound nice and solve some of the problems I’ve mentioned here. However, since I have about 37,000 lines of BibTeX sitting around, I’m not interested in migrating to a new system until a clear winner emerges.
