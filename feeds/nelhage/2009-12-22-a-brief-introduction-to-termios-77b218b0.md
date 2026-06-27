---
title: A Brief Introduction to termios
url: https://blog.nelhage.com/2009/12/a-brief-introduction-to-termios/
published: "2009-12-22T19:11:22Z"
feed: nelhage
guid: https://blog.nelhage.com/2009/12/a-brief-introduction-to-termios/
---

# A Brief Introduction to termios

If you’re a regular user of the terminal on a UNIX system, there are probably a large number of behaviors you take mostly for granted without really thinking about them. If you press ^C or ^Z it kills or stops the foreground program – unless it’s something like emacs or vim, in which case it gets handled like a normal keystroke. When you ssh to a remote host, though, they go to the processes on that machine, not the ssh process.
