---
title: 'New reptyr feature: TTY-stealing'
url: https://blog.nelhage.com/2014/08/new-reptyr-feature-tty-stealing/
published: "2014-08-20T08:41:34Z"
feed: nelhage
guid: https://blog.nelhage.com/2014/08/new-reptyr-feature-tty-stealing/
---

# New reptyr feature: TTY-stealing

Ever since I wrote reptyr, I’ve been frustrated by a number of issues in reptyr that I fundamentally didn’t know how to solve within the reptyr model. Most annoyingly, reptyr fundamentally only worked on single processes, and could not attach processes with children, making it useless in a large class of real-world situations. TTY stealing Recently, I merged an experimental reptyr feature that I call “tty-stealing”, which has the potential to fix all of these issues (with some other disadvantages, which I’ll discuss later).
