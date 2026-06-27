---
title: todo.pl ratmenu
url: https://blog.nelhage.com/2008/02/todopl-ratmenu/
published: "2008-02-19T23:46:00Z"
feed: nelhage
guid: https://blog.nelhage.com/2008/02/todopl-ratmenu/
---

# todo.pl ratmenu

broder has been hacking on some better quicksilver integration for Hiveminder using todo.pl. I don’t use a mac, but I don’t see why linux users shouldn’t get fun toys to. So I hacked up the following two-liner that uses todo.pl and ratmenu to pop up a list of tasks, and mark one as completed: #!/bin/sh todo.pl \| perl -ne 'push @a,$2,"todo.pl done $1" if /^#(\[\\w\]+) (.+)$/;' \ -e 'END{exec("ratmenu",@a)}' I dropped it into my ~/bin and bound it to C-t x in my window manager (XMonad).
