---
title: Property Testing Like AFL
url: https://blog.nelhage.com/post/property-testing-like-afl/
published: "2017-10-24T16:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/property-testing-like-afl/
---

# Property Testing Like AFL

In my last last post, I argued that property-based testing and fuzzing are essentially the same practice, or at least share a lot of commonality. In this followup post, I want to explore that idea a bit more: I’ll first detour into some of my frustrations and hesitations around typical property-based testing tools, and then propose a hypothetical UX to resolve these concerns, which takes heavy inspiration from modern fuzzing tools, specifically the AFL and Google’s OSS-Fuzz.
