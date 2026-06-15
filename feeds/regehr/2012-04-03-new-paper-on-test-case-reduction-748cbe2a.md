---
title: New Paper on Test-Case Reduction
url: https://blog.regehr.org/archives/699
published: "2012-04-03T05:25:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=699
---

# New Paper on Test-Case Reduction

For about the last four years my group has struggled with the problem of taking a large program that makes a compiler do something bad (crash, run forever, generate incorrect code) and turning it into a minimum-sized program that elicits the same bad behavior. [Delta](http://delta.tigris.org/) is a great tool but it does not entirely solve the problem. Anyway, the short version of the story is that we spent a ton of time on this problem and only made good headway during the last year or so.

Our various approaches and their results are written up in a [paper](http://www.cs.utah.edu/~regehr/papers/pldi12-preprint.pdf) that I’ll present at PLDI 2012 in Beijing in June. I’m pretty happy with it. The audience is narrow but one of the new tools, C-Reduce (we’ll announce a release here shortly), really nails the problem and produces output far smaller than any tool anyone else has produced.

It had been a long time (eight years, perhaps) since I took both the technical lead and writing lead on a paper and let me tell you, it was awful. I’m not saying I did all the work or anything, but I did a lot of it and it about killed me. Anyway, I have a fresh appreciation for implementation effort that students put into a systems paper. C-Reduce is a simple tool that has basically one internal interface, and somehow it took at least a dozen iterations before I arrived at a version of this interface that I’m happy with.
