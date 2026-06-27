---
title: amd64 and va_arg
url: https://blog.nelhage.com/2010/10/amd64-and-va_arg/
published: "2010-10-04T00:14:28Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/10/amd64-and-va_arg/
---

# amd64 and va_arg

A while back, I was poking around LLVM bugs, and discovered, to my surprise, that LLVM doesn’t support the va\_arg intrinsic, used by functions to accept multiple arguments, at all on amd64. It turns out that clang and llvm-gcc, the compilers that backend to LLVM, have their own implementations in the frontend, so this isn’t as big a deal as it might sound, but it was still a surprise to me.
