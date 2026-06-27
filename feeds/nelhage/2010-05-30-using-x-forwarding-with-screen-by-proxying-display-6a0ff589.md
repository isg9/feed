---
title: Using X forwarding with screen by proxying $DISPLAY
url: https://blog.nelhage.com/2010/05/using-x-forwarding-with-screen/
published: "2010-05-30T20:25:52Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/05/using-x-forwarding-with-screen/
---

# Using X forwarding with screen by proxying $DISPLAY

If you’re reading this blog, I probably don’t have to explain why I love GNU screen. I can keep a long-running session going on a server somewhere, and log in and resume my session without losing any state. I also love X-forwarding. I love being able to log into a remote server and work in a shell there, but still pop up graphical windows (for instance, gitk’s) on my local machine when I need to.
