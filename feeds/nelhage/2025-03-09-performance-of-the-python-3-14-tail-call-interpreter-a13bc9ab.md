---
title: Performance of the Python 3.14 tail-call interpreter
url: https://blog.nelhage.com/post/cpython-tail-call/
published: "2025-03-09T22:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/cpython-tail-call/
---

# Performance of the Python 3.14 tail-call interpreter

About a month ago, the CPython project merged a new implementation strategy for their bytecode interpreter. The initial headline results were very impressive, showing a 10-15% performance improvement on average across a wide range of benchmarks across a variety of platforms. Unfortunately, as I will document in this post, these impressive performance gains turned out to be primarily due to inadvertently working around a regression in LLVM 19. When benchmarked against a better baseline (such GCC, clang-18, or LLVM 19 with certain tuning flags), the performance gain drops to 1-5% or so depending on the exact setup.
