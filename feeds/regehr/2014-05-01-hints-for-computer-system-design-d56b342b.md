---
title: Hints for Computer System Design
url: https://blog.regehr.org/archives/1143
published: "2014-05-01T19:00:34Z"
feed: regehr
guid: http://blog.regehr.org/?p=1143
---

# Hints for Computer System Design

On the last day of my advanced OS course this spring we discussed one of my all-time favorite computer science papers: Butler Lampson’s [Hints for Computer System Design](http://www.stanford.edu/class/cs240/readings/lampson-hints.pdf). Why is it so good?

- It’s hard-won advice. Designing systems is not easy — a person can spend a lifetime learning to do it well — and any head start we can get is useful.

- The advice is broadly applicable and is basically correct, there aren’t really any places where I need to tell students “Yeah… ignore Section 3.5.”

- There are many, many examples illustrating the hints. Some of them require a bit of historical knowledge (the paper is 30 years old) but by and large the examples stand the test of time.

- It seems to me that a lot of Lampson’s hints were routinely ignored by the developers of large codes that we use today. I think the reason is pretty obvious: the massive increases in throughput and storage capacity over the last 30 years have permitted a great deal of sloppy code to be created. It’s nice to read a collection of clear thinking about how things could be done better.

Something that bums me out is that it’s now impossible to publish a paper like this at a top conference such as SOSP.
