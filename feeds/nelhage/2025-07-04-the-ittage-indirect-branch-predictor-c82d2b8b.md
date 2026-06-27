---
title: The ITTAGE indirect branch predictor
url: https://blog.nelhage.com/post/ittage-branch-predictor/
published: "2025-07-04T21:30:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/ittage-branch-predictor/
---

# The ITTAGE indirect branch predictor

While investigating the performance of the new Python 3.14 tail-calling interpreter, I learned (via this very informative comment from Sam Gross) new (to me) piece of performance trivia: Modern CPUs mostly no longer struggle to predict the bytecode-dispatch indirect jump inside a “conventional” bytecode interpreter loop. In steady-state, assuming the bytecode itself is reasonable stable, modern CPUs achieve very high accuracy predicting the dispatch, even for “vanilla” while / switch-style interpreter loops1!
