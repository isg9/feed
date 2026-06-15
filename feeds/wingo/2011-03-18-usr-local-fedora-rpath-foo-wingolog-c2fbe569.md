---
title: /usr/local, fedora, rpath, foo — wingolog
url: https://wingolog.org/archives/2011/03/18/usrlocal-fedora-rpath-foo
published: "2011-03-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/03/18/usrlocal-fedora-rpath-foo
---

# /usr/local, fedora, rpath, foo — wingolog

## [/usr/local, fedora, rpath, foo](/archives/2011/03/18/usrlocal-fedora-rpath-foo)

18 March 2011 11:52 PM

- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [fedora](/tags/fedora)
- [compilation](/tags/compilation)
- [bugs](/tags/bugs)

Good evening intertubes,

I wish to broach a nerdy topic, once more. May my lay readers be rewarded in some heaven for your earthly sufferings. I'll see you all at the bar.

**problem statement**

For the rest of us, we have a topic, and that topic is about autotools, software installation, `-rpath`, and all that kind of thing.

As you might be aware, last month we released [Guile](http://www.gnu.org/software/guile/) 2.0.0. Of course we got a number of new interested users. For some of them things went great. It all started like this:

```
wget http://ftp.gnu.org/gnu/guile/guile-2.0.0.tar.gz
tar zxvf guile-2.0.0.tar.gz
cd guile-2.0.0
./configure
make
sudo make install

```

Up to here, things are pretty much awesome for everybody. Obviously the next thing is to run it.

```
$ guile
GNU Guile 2.0.0
Copyright (C) 1995-2011 Free Software Foundation, Inc.

Guile comes with ABSOLUTELY NO WARRANTY; for details type `,show w'.
This program is free software, and you are welcome to redistribute it
under certain conditions; type `,show c' for details.

Enter `,help' for help.
scheme@(guile-user)> "Hello, World!"
$1 = "Hello, World!"

```

Sweetness! Let's start going through the manual. [(Time passes.)](http://www.gnu.org/software/guile/manual/) You get to the point of compiling a short program that links to Guile:

```
gcc -o simple-guile simple-guile.c \
  `guile-config compile` `guile-config link`

```

And so far so good! But alack:

```
$ ./simple-guile
./simple-guile: error while loading shared libraries:
    libguile-2.0.so.22: cannot open shared object

```

Pants! What is the deal?

**righteous indignation**

The deal is, you appear to be on a Fedora or some other Red Hat-derived system. These systems add `/usr/local/bin` to the `PATH`, so the `guile-config` call succeeds. `/usr/local/lib` is in the link-time path too, so it finds `libguile-2.0.so`. But `/usr/local/lib` is *not* in the *runtime* library lookup path by default. Hence the error above.

This, my friends, is a bug. It is a bug in Fedora and similar systems. This recipe works fine on Debian. It works fine with the GNU toolchain, as configured out-of-the-box.

The only reason I can think that Fedora would break this is because of their `lib` versus `lib64` split. It would be strange to say, "32-bit libraries go in `/usr/lib`, but maybe or maybe not in `/usr/local/lib`." And in fact the FHS explicitly disclaims what's happening in `/usr/local`.

But the fact is, `/usr/local` is the default location for people to compile, and the `$PATH` and other settings indicate that Fedora folk know this. I can only conjecture that the situation is this way in order to preserve compatibility with legacy 32-bit closed-source binary apps that cannot be recompiled to politely install their libraries elsewhere.

This decision costs me time and energy in the form of bug reports. Thanks but no thanks, whoever made this decision.

**solutions?**

It's true, this is a specific case of a more general issue. If you installed instead to `/opt/guile`, you would have to adjust your `PATH` to be able to run `guile-config`, or your `PKG_CONFIG_PATH` if you wanted to use the underlying `pkg-config` file instead. So you do that:

```
export PKG_CONFIG_PATH=/opt/guile/lib/pkgconfig
gcc -o simple-guile simple-guile.c `pkg-config --cflags --libs guile-2.0`

```

Awesome. You run it:

```
$ ./simple-guile

```

Hey awesome it works! Actually that's not true. Or rather, it's only true if you're on some other system, like a Mac. If you're on GNU, you get, again:

```
./simple-guile: error while loading shared libraries:
   libguile-2.0.so.22: cannot open shared object

```

Willikers! The same badness! And in this case, we can't even blame Fedora, as no one would think to add `/opt/guile/lib` to the runtime library path. (Well, actually, Apple did; it linked the binary to the link-time location of the library. That has other problems, though.)

The deal is, that we ourselves need to add `/opt/guile/lib` to the runtime library lookup path. But how we do that is compiler-specific; and in any case it's not necessary if we're installing to somewhere that's already in the system's runtime search path.

So---and this is the point I was coming to, besides ripping on Fedora's defaults---you need a build-time determination of what to add to the compilation line to set the rpath.

That, my friends, is the function of [AC\_LIB\_HAVE\_LINKFLAGS](http://www.gnu.org/software/gnulib/manual/html_node/Searching-for-Libraries.html), from the excellent gnulib. It takes some link-time flags and spits out something that will also set the run-time library search path.

**why haven't I seen this?**

Amusingly I had not personally encountered this issue because all of my projects use libtool for linking, which does add `-rpath` as appropriate. Libtool-induced blissful ignorance: who'd have thought it possible?

Having said my message, I now entreat my more informed readers to leave corrections below in the comments. Thanks!

## related articles

- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)
- [ephemeral success](/archives/2022/12/15/ephemeral-success)
- [i'm throwing ephemeron party & you're invited](/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited)
- [we iterate so that you can recurse](/archives/2022/12/11/we-iterate-so-that-you-can-recurse)

### 4 responses

1. Anonymous says:[19 March 2011 5:40 PM](#9cf62a50ac7a36f880d3fed33638f751de1c5bf6)

   Putting /usr/local/lib in the runtime library search path leads to a different problem, though: it means libraries in /usr/local/lib can potentially override system libraries for use by packaged binaries. That leads to very confused bug reports by users, who encounter strange issues running packaged binaries, and end up finding stray libraries in /usr/local/lib causing the problem.

2. Ralf Wildenhues says:[21 March 2011 4:26 AM](#319dff6b1c31586fd9f79888b6fc2e6e6ec062cb)

   Who leaves "stray" libraries in /usr/local/lib? Or put another way, admin errors can always make your life hard.

   That said, the issue is similar with /opt/lib or other extension directories that are not in the default search path.

   One problem with the -rpath hardcoding which libtool does is that it doesn't play so well when relocating installed packages, and when cross compiling for a sysrooted system (but see the --with-sysroot support in Libtool 2.4).

3. [Debarshi Ray](http://debarshiray.wordpress.com/) says:[6 May 2011 0:08 AM](#b2aa29e8a5516533f5fef47109d9ad72d7de9284)

   Instead of /usr/local, I put all my stuff in /opt. Mainly because in Fedora /usr/local has all sorts of directories already created and I really love the ability to rm -rf \* my $prefix without feeling that I may have deleted something that was present in the distribution by default.

   Anyway.

   For run-time linking I have created a /etc/ld.so.conf.d/local.conf with the following:

   /opt/lib

   And in ~/.bash\_profile I have got:

   export PKG\_CONFIG\_PATH=/opt/lib/pkgconfig

   For gtk-doc documentation I create the appropriate links under /home/rishi/.local/share/gtk-doc/html.

   Ofcourse this does not take care of DBus services, so I manually start the DBus service if I am going to test one that I installed in /opt.

4. Tristan Matthews says:[22 March 2013 3:57 PM](#41bfbd04192b08cdab77512775fbc16d991e6134)

   Great article!

   Just in passing, for uninstalled D-Bus services, you can simply add a /etc/dbus-1/session.d/org.SERVICENAME.USER.conf file.

   We have an example here:

   https://projects.savoirfairelinux.com/projects/sflphone/wiki/How\_to\_build#Notes-on-installation-outside-of-usr-or-usrlocal

Comments are closed.
