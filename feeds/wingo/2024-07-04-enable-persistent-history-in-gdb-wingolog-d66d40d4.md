---
title: enable persistent history in gdb — wingolog
url: https://wingolog.org/archives/2024/07/04/enable-persistent-history-in-gdb
published: "2024-07-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/07/04/enable-persistent-history-in-gdb
---

# enable persistent history in gdb — wingolog

## [enable persistent history in gdb](/archives/2024/07/04/enable-persistent-history-in-gdb)

4 July 2024 10:10 AM

- [gdb](/tags/gdb)
- [command line](/tags/command%20line)
- [tui](/tags/tui)
- [readline](/tags/readline)

Friends. I have been using GDB for more than two decades and have been annoyed by the fact that, unlike the shell, it doesn’t keep a persistent history.

Of course, it has always been able to do that, but history saving is just not on by default. So do yourself a favor and turn it on by pasting this into your terminal:

```
mkdir -p ~/.config/gdb
mkdir -p ~/.cache/gdb
echo 'set history filename ~/.cache/gdb/history' >> ~/.config/gdb/gdbinit
echo 'set history save on' >> ~/.config/gdb/gdbinit
echo 'set history size unlimited' >> ~/.config/gdb/gdbinit

```

You are welcome!

## related articles

- [libtool + gdb, a cure for pain](/archives/2008/10/15/libtool-gdb-a-cure-for-pain)

### One response

1. Jan Alexander Steffens says:[5 July 2024 0:54 AM](#12365cacea7195c1a723aefeac5d8ae983304018)

   You could also use a line ‘shell mkdir -p -m 0700 ~/.cache/gdb’ in your gdbinit to ensure the cache directory exists.

   Might be helpful if you’re using a mechanism to place the config file but not necessarily the cache dir, or if ~/.cache gets pruned.

Comments are closed.
