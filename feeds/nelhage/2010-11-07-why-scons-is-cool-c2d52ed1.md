---
title: Why scons is cool
url: https://blog.nelhage.com/2010/11/why-scons-is-cool/
published: "2010-11-07T18:00:38Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/11/why-scons-is-cool/
---

# Why scons is cool

I’ve recently started playing with scons a little for some small personal projects. It’s not perfect, but I’ve rapidly come to the conclusion that it’s a probably far better choice than make in many cases. The main exceptions would be cases where you need to integrate into legacy build systems, or if asking or expecting developers to have scons installed is unreasonable for some reason. The main reason that scons is cool to me, and the thing that makes it fundamentally different from make, is the introduction of actual scoping.
