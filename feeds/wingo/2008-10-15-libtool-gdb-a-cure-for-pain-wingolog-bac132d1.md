---
title: libtool + gdb, a cure for pain — wingolog
url: https://wingolog.org/archives/2008/10/15/libtool-gdb-a-cure-for-pain
published: "2008-10-15T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/10/15/libtool-gdb-a-cure-for-pain
---

# libtool + gdb, a cure for pain — wingolog

## [libtool + gdb, a cure for pain](/archives/2008/10/15/libtool-gdb-a-cure-for-pain)

15 October 2008 9:27 AM

- [libtool](/tags/libtool)
- [gdb](/tags/gdb)
- [hack](/tags/hack)

I finally got tired of misinchanting " `gdb mumble .libs lt --args foo bar mumble`" and dropped this into my `~/bin` as `lgdb`:

```
#!/bin/bash
prog=$1
shift
exec gdb --args $(dirname $prog)/.libs/lt-$(basename $prog) "$@"

```

I am aware of the *`libtool gdb`* idiom, but that appears to have been deprecated. There is a current of thought within autotools people that abhors concision. I do not understand it.

(As long as I'm ranting, I wish that libtool would make the shell script + relinking case secondary to the binary case, if it is at all necessary.)

I hope this post is as superfluous as [the last](//wingolog.org/archives/2008/10/14/merging-in-unrelated-git-branches), for the world if not for my ego.

## related articles

- [enable persistent history in gdb](/archives/2024/07/04/enable-persistent-history-in-gdb)
- [sign of the times](/archives/2021/04/08/sign-of-the-times)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [the good hack](/archives/2009/07/26/the-good-hack)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [introducing griddy](/archives/2008/07/31/introducing-griddy)

### 8 responses

1. James Henstridge says:[15 October 2008 9:53 AM](#c7e8e3390b5d3c7b1f136dc97148dffc46711f39)

   The following should work: "libtool --mode=execute gdb programname". This is what "libtool gdb" was shorthand for, and I'd be surprised if they removed the --mode=execute feature.

   Given that it acts a a pure prefix, you could use it as an alias rather than a shell script.

2. lamby says:[15 October 2008 1:12 PM](#faba00e02002149be60de4a65cc765e02e16c322)

   That's not a shell script, it's a Bash script.

3. Dan Winship says:[15 October 2008 1:23 PM](#7e52e8715fd9d19df590c66ab0f490f8c7894f06)

   \> (As long as I'm ranting, I wish that libtool would make the shell script + relinking case secondary to the binary case, if it is at all necessary.)

   It's necessary if you're building a library that you already have installed, because if you don't set LD\_LIBRARY\_PATH, the newly-built program will dynamically link against /usr/lib/libmumble.so rather than ../libmumble/.libs/libmumble.so like you wanted.

   Then again, maybe if you build with --enable-maintainer-mode, it could just build the working-directory copy with the mangled library path automatically, and then relink it with the standard library path only when you do "make install"...

   Anyway, knowing the "libtool --mode=execute" idiom is useful because you can use it with things other than gdb too (strace, valgrind, etc).

4. [Andy Wingo](http://wingolog.org/) says:[15 October 2008 1:58 PM](#fae35a56c62993e20caf052d2240dc2b52ca451a)

   James, you're right as usual -- I should probably use that instead of poking .libs. It will also take care of making sure that the lt-foo is there, as it is not there until it is first run.

   Dan, it seems that once the lt-foo executable is created, it is indeed linked to the in-tree libs. So there does not actually seem to be a need for the shell script at all, given that libtool controls the installation process and can relink then anyway. (AM\_MAINTAINER\_MODE is sortof-deprecated too, in case you hadn't heard.)

   Lamby. Picking nits is entertaining but you must take more care. Your statement is vague ("That"? I assume you refer to my script?), incorrect (Bash is indeed a shell), and misguided (I never called it a shell script).

5. [Benoît Dejean](http://www.placenet.org/benoit) says:[15 October 2008 2:18 PM](#f82800619cfd99c2262db16310c08101453174d5)

   Or use Nemiver !

6. Baptiste says:[16 October 2008 5:02 AM](#4ecd7e43e2b25f34709e875d9669e7981ec1e11a)

   Yep, Nemiver supports libtool wrapper in latest release.

7. [Will Thompson](http://willthompson.co.uk/) says:[17 October 2008 11:01 AM](#10be1c3611736e5f3c06dcb65da53ae8358c7f9c)

   Andy: I think Lamby was picking James' nits, not yours. :)

8. [Andy Wingo](http://wingolog.org/) says:[17 October 2008 1:28 PM](#0c63756b53feb113b901eaebe69ea9ad2d9cba88)

   Will: Are you picking my nits? ;-)

Comments are closed.
