---
title: 1500+ Bugs from One Fuzzer
url: https://blog.regehr.org/archives/731
published: "2012-06-18T20:08:17Z"
feed: regehr
guid: http://blog.regehr.org/?p=731
---

# 1500+ Bugs from One Fuzzer

![](https://blog.regehr.org/wp-content/uploads/2012/06/amazing-lady-bug-swarm-preview.jpg)

[This metabug](https://bugzilla.mozilla.org/show_bug.cgi?id=349611) links to all of the defects found in Firefox’s JavaScript engine by [jsfunfuzz](http://www.squarefree.com/2007/08/02/introducing-jsfunfuzz/). The surprise here isn’t that bugs were found, but rather that more than 1500 bugs were found in a single language runtime by a single test case generator. I’m interested in exactly what is going on here. One possibility would be that JS performance has become so important in the last five years that it supersedes all other goals, making these bugs inevitable.  Another possibility is that something is wrong with the architecture or the development process of this particular JS engine. It’s also possible that I’m simply out of touch with bug rates in real software development efforts and this kind of result is perfectly normal and expected. Regrettably, jsfunfuzz is no longer public, most likely because it was like handing a loaded gun to the enemy. Anyhow, jsfunfuzz serves as an excellent example of how powerful random testing can be.
