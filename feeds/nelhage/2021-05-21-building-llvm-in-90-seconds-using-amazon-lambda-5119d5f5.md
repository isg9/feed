---
title: Building LLVM in 90 seconds using Amazon Lambda
url: https://blog.nelhage.com/post/building-llvm-in-90s/
published: "2021-05-21T02:00:28Z"
feed: nelhage
guid: https://blog.nelhage.com/post/building-llvm-in-90s/
---

# Building LLVM in 90 seconds using Amazon Lambda

Last week, Frederic Cambus wrote about building LLVM quickly on some very large machines, culminating in a 2m37s build on a 160-core ARM machine. I don’t have a giant ARM behemoth, but I have been working on a tool I call Llama, which lets you offload computational work – including C and C++ builds – onto Amazon Lambda. I decided to see how good it could do at a similar build.
