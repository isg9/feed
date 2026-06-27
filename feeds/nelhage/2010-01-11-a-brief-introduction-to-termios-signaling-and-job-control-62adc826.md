---
title: 'A Brief Introduction to termios: Signaling and Job Control'
url: https://blog.nelhage.com/2010/01/a-brief-introduction-to-termios-signaling-and-job-control/
published: "2010-01-11T01:42:52Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/01/a-brief-introduction-to-termios-signaling-and-job-control/
---

# A Brief Introduction to termios: Signaling and Job Control

(This is part three of a multi-part introduction to termios and terminal emulation on UNIX. Read part 1 or part 2 if you’re new here) For my final entry on termios, I will be looking at job control in the shell (i.e. backgrounding and foreground jobs) and the very closely related topic of signal generation by termios, in response to INTR and friends. Sessions and Process Groups For the purposes of termios, processes are organized into two hierarchical groups, process groups and sessions.
