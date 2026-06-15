---
title: 'Wanted: Epitaphs for Hot Topics'
url: https://blog.regehr.org/archives/667
published: "2012-01-18T23:40:51Z"
feed: regehr
guid: http://blog.regehr.org/?p=667
---

# Wanted: Epitaphs for Hot Topics

Any given research community always has a few hot topics that attract an inordinate number of paper submissions. Sometimes these are flashes in the pan, other times they mature into full-fledged areas having their own workshops and such — but most often they endure for a few years, result in a pile of PhDs, and then slowly die off. After this happens I often find myself with unanswered questions:

- Were the underlying research problems solved or were they too hard?
- Did the solutions migrate into practice or are they withering on the academic vine?
- What lessons were learned?

For example, consider [software-based distributed shared memory](http://en.wikipedia.org/wiki/Distributed_shared_memory), where MMU tricks are used to provide a shared address space for a cluster of machines. My former colleague [John Carter](http://scholar.google.com/citations?user=bjCTmeMAAAAJ&hl=en) wrote some of the basic papers in this area and they are hugely cited: more than 2000 citations for just the top four papers (including the patent). Clearly, a lot of followup papers were written. But then, as far as I know, software DSM is not widely used today and certainly it’s not an area where papers are being written. My understanding (which could easily be flawed) is that DSM just abstracted away a few too many things, such as performance and node failures, and this made DSM hard to use compared to explicit messaging. On the other hand, the DSM community created a detailed understanding of memory coherence models and (I believe) this directly fed into the formalization of the Java and C++ memory models. Thus, the practical impact of this research area is large, even if a bit oblique.

Couple more examples:

- [This short thread on G+](https://plus.google.com/109706986708019547706/posts/RdEXUXnN69R) where Matt Welsh asks: “What happened to DHTs?” The thread works because the right people responded — it’s really interesting reading.
- Blog post: [Was P2P live streaming an academic bubble?](http://peerdal.blogspot.com/2011/10/was-p2p-live-streaming-academic-bubble.html)

My experience is that the people who were there — writing papers about DSM or DHTs or P2P or whatever — always know the answers to my questions above. The problem is, by the time the answers become apparent everyone is working on the next hot topic and going up for tenure or whatever, and has zero time to write a proper retrospective. Furthermore, in many cases an honest retrospective would need to be a bit brutal, for example to indicate which papers really just were not good ideas (of course some of these will have won “best paper” awards). In the old days, these retrospectives would have required a venue willing to publish them, perhaps ACM Computing Surveys or similar, but today they could be uploaded to arXiv. I would totally read and cite these papers if they existed, and would make my students do the same.

**Updates:**

If you know of a good research epitaph, please send a pointer.

- Michael Hind’s paper [Pointer Analysis: Haven’t We Solved This Problem Yet?](http://www.cs.cornell.edu/courses/cs711/2005fa/papers/hind-paste01.pdf) isn’t quite what I’m looking for, but it’s not too far off either.
- David Andersen sent a link to this excellent [epitaph for QoS](http://www.cl.cam.ac.uk/~jac22/out/ripqos-rant.pdf).
- Bhaskar Krishnamachari sent a pointer to [this paper](http://www.cl.cam.ac.uk/~sos22/p50-kurkowski.pdf) about the state of simulation studies in wireless ad hoc networks; it’s not really an epitaph but serves a similar role — capturing lessons that need to be learned.
