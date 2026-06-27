---
title: Measuring Capacity Through Utilization
url: https://blog.nelhage.com/post/utilization/
published: "2017-01-08T20:09:09Z"
feed: nelhage
guid: https://blog.nelhage.com/post/utilization/
---

# Measuring Capacity Through Utilization

(This post is cross-posted from Honeycomb’s instrumentation series). One of my favorite concepts when thinking about instrumenting a system to understand its overall performance and capacity is what I call “time utilization”. By this I mean: If you look at the behavior of a thread over some window of time, what fraction of its time is spent in each “kind” of work that it does? Let’s make this notion concrete by examining a toy example.
