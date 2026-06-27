---
title: 'A Brief Introduction to termios: termios(3) and stty'
url: https://blog.nelhage.com/2009/12/a-brief-introduction-to-termios-termios3-and-stty/
published: "2009-12-30T01:47:17Z"
feed: nelhage
guid: https://blog.nelhage.com/2009/12/a-brief-introduction-to-termios-termios3-and-stty/
---

# A Brief Introduction to termios: termios(3) and stty

(This is part two of a multi-part introduction to termios and terminal emulation on UNIX. Read part 1 if you’re new here) In this entry, we’ll look at the interfaces that are used to control the behavior of the “termios” box sitting between the master and slave pty. The behaviors I described last time are fine if you have a completely dumb program talking to the terminal, but if the program over on the right is using curses (like emacs or vim), or even just readline (like bash), it will want to disable or customize some of the behaviors.
