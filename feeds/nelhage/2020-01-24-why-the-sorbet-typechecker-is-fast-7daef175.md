---
title: Why the Sorbet typechecker is fast
url: https://blog.nelhage.com/post/why-sorbet-is-fast/
published: "2020-01-24T01:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/why-sorbet-is-fast/
---

# Why the Sorbet typechecker is fast

This is the second in an indefinite series of posts about things that I think went well in the Sorbet project. The previous one covered our testing approach. Sorbet is fast. Numerous of our early users commented specifically on how fast it was, and how much they appreciated this speed. Our informal benchmarks on Stripe’s codebase clocked it as typechecking around 100,000 lines of code per second per core, making it one of the fastest production typecheckers we are aware of.
