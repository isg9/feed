---
title: From error-handling to structured concurrency
url: https://blog.nelhage.com/post/concurrent-error-handling/
published: "2026-03-23T15:30:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/concurrent-error-handling/
---

# From error-handling to structured concurrency

How should we think about error-handling in concurrent programs? In single-threaded programs, we’ve mostly converged on a standard pattern, with a diverse zoo of implementations and concrete patterns. When an error occurs, it is propagated up the stack until we find a stack frame which is prepared to handle it. As we do so, we unwind the stack frames in-order, giving each frame the opportunity to clean up or destroy resources as appropriate.
