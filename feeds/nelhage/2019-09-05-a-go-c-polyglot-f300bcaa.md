---
title: A Go/C Polyglot
url: https://blog.nelhage.com/post/a-go-c-polyglot/
published: "2019-09-05T23:42:28Z"
feed: nelhage
guid: https://blog.nelhage.com/post/a-go-c-polyglot/
---

# A Go/C Polyglot

Writing a Go/C polyglot Someone on a Slack I’m on recently raised the question of how you might write a source file that’s both valid C and Go, commenting that it wasn’t immediately obvious if this was even possible. I got nerdsniped, and succeeded in producing one, which you can find here. I’ve been asked how I found that construction, so I thought it might be interesting to document the thought / discovery / exploration process that got me there.
