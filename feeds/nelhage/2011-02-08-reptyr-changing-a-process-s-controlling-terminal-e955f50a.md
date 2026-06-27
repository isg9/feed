---
title: 'reptyr: Changing a process''s controlling terminal'
url: https://blog.nelhage.com/2011/02/changing-ctty/
published: "2011-02-08T23:06:50Z"
feed: nelhage
guid: https://blog.nelhage.com/2011/02/changing-ctty/
---

# reptyr: Changing a process's controlling terminal

reptyr (announced recently on this blog) takes a process that is currently running in one terminal, and transplants it to a new terminal. reptyr comes from a proud family of similar hacks, and works in the same basic way: We use ptrace(2) to attach to a target process and force it to execute code of our own choosing, in order to open the new terminal, and dup2(2) it over stdout and stderr.
