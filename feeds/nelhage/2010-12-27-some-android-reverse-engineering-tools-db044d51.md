---
title: Some Android reverse-engineering tools
url: https://blog.nelhage.com/2010/12/some-android-reverse-engineering-tools/
published: "2010-12-27T16:26:13Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/12/some-android-reverse-engineering-tools/
---

# Some Android reverse-engineering tools

I’ve spent a lot of time this last week staring at decompiled Dalvik assembly. In the process, I created a couple of useful tools that I figure are worth sharing. I’ve been using dedexer instead of baksmali, honestly mainly because the former’s output has fewer blank lines and so is more readable on my netbook’s screen. Thus, these tools are designed to work with the output of dedexer, but the formats are simple enough that they should be easily portable to smali, if that’s your tool of choice (And it does look like a better tool overall, from what I can see).
