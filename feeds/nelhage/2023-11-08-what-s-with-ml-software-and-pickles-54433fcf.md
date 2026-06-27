---
title: What's with ML software and pickles?
url: https://blog.nelhage.com/post/pickles-and-ml/
published: "2023-11-08T05:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/pickles-and-ml/
---

# What's with ML software and pickles?

I have spent many years as an software engineer who was a total outsider to machine-learning, but with some curiosity and occasional peripheral interactions with it. During this time, a recurring theme for me was horror (and, to be honest, disdain) every time I encountered the widespread usage of Python pickle in the Python ML ecosystem. In addition to their major security issues1, the use of pickle for serialization tends to be very brittle, leading to all kinds of nightmares as you evolve your code and upgrade libraries and Python versions.
