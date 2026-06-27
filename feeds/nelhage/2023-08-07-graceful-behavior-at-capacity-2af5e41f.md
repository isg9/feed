---
title: Graceful behavior at capacity
url: https://blog.nelhage.com/post/systems-at-capacity/
published: "2023-08-07T16:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/systems-at-capacity/
---

# Graceful behavior at capacity

Suppose we’ve got a service. We’ll gloss over the details for now, but let’s stipulate that it accepts requests from the outside world, and takes some action in response. Maybe those requests are HTTP requests, or RPCs, or just incoming packets to be routed at the network layer. We can get more specific later. What can we say about its performance? All we know is that it receives requests, and that it acts on them.
