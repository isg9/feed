---
title: 'reptyr: Attach a running process to a new terminal'
url: https://blog.nelhage.com/2011/01/reptyr-attach-a-running-process-to-a-new-terminal/
published: "2011-01-21T21:56:01Z"
feed: nelhage
guid: https://blog.nelhage.com/2011/01/reptyr-attach-a-running-process-to-a-new-terminal/
---

# reptyr: Attach a running process to a new terminal

Over the last week, I’ve written a nifty tool that I call reptyr. reptyr is a utility for taking an existing running program and attaching it to a new terminal. Started a long-running process over ssh, but have to leave and don’t want to interrupt it? Just start a screen, use reptyr to grab it, and then kill the ssh session and head on home. You can grab the source, or read on for some more details.
