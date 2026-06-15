---
title: a preview look at quagmire — wingolog
url: https://wingolog.org/archives/2008/04/28/a-preview-look-at-quagmire
published: "2008-04-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/28/a-preview-look-at-quagmire
---

# a preview look at quagmire — wingolog

## [a preview look at quagmire](/archives/2008/04/28/a-preview-look-at-quagmire)

28 April 2008 3:57 PM

- [autotools](/tags/autotools)
- [automake](/tags/automake)
- [quagmire](/tags/quagmire)
- [tromey](/tags/tromey)
- [build systems](/tags/build%20systems)
- [workflow](/tags/workflow)

We've been using git at work for a few months now, and finally settling down reasonable workflow. As reasonable as anything might be with a 9 hour difference between me and Los Angeles, and for a cross-platform project that uses autotools.

Regarding the latter, the autotools have been a constant source of pain for my co-workers. I don't mind it myself, but I feel bad because I had a hand in inflicting it on them. So now when hacking on a bit of test code, I decided to take a look at [Quagmire](http://code.google.com/p/quagmire), Tom Tromey's experiment in replacing automake, libtool, and (to a large degree) autoconf with portable GNU make.

With Quagmire, all of the autofoo is replaced with one makefile, `Quagmire.mk`. Its format is much like that of a `Makefile.am`. A simple project with a few source files, that depends on pkg-config, might look like this example, taken from the Quagmire distribution:

```
# this is Quagmire.mk, in the top-level source directory
# -*- makefile-gmake -*-

bin_PROGRAMS = ekeyring

lib_LIBRARIES = libzardoz.a

libzardoz.a_SOURCES = zardoz.c

ekeyring_SOURCES = ekeyring.c something.h
ekeyring_PACKAGES = gnome-keyring-1
ekeyring_CONFIG_HEADERS = config.h

config.h_FUNCTIONS = strcmp strdup
config.h_HEADERS = string.h nothing.h strings.h sys/types.h

data_SCRIPTS = ekeyring.c
data_DATA = something.h

EXTRA_DIST = README.simple

```

The syntax will be familiar to anyone who has used automake. It is further described in the [README](http://code.google.com/p/quagmire/source/browse/trunk/README?r=61).

Then you add the following configure.ac to the same directory:

```
AC_INIT(ekeyring, 1.5.1)
AC_CONFIG_SRCDIR(ekeyring.c)
AC_ARG_PROGRAM
AM_QUAGMIRE
AC_OUTPUT

```

And copy quagmire.m4 (from [http://quagmire.googlecode.com/svn/trunk/m4/quagmire.m4](http://quagmire.googlecode.com/svn/trunk/m4/quagmire.m4)) to aclocal.m4, to provide the AM\_QUAGMIRE definition. Then import the quagmire files into your source tree by running:

```
svn checkout http://quagmire.googlecode.com/svn/trunk/src quagmire

```

At this point you run autoconf, in order to produce `configure`, and then ./configure. It doesn't configure much. Most of the actual configure checks are run at make-time.

For example the `ekeyring_PACKAGES` line specifies the pkg-config files that are needed to compile ekeyring. Those checks are made as dependencies of the ekeyring target.

**status**

Quagmire is promising, and is quite fast, as is appropriate for an underfeatured project. It's not there yet though. It doesn't yet understand differing shared library extensions (.so versus .dylib), it doesn't (I don't think) know about static linking like libtool's `.la` files do, and I haven't gotten programs that depend on in-tree dynamic or static libraries to build yet.

And yet, it does a proper distcheck. It integrates with pkg-config. The makefiles are understandable -- I can get the whole process in my head. Tom Tromey is either exactly the person to fix autotools, considering that he wrote automake, or exactly the wrong person, depending on your religion. And GNU make is actually [quite an OK language](http://okmij.org/ftp/Computation/Computation.html#Makefile-functional) for expressing functional dependencies.

For now I'm going to hold off on trying to use Quagmire for a while at work, given that I need to do some static linking stuff on the mac. But it's an idea that will be in the back of my mind.

### 2 responses

1. [Jeff Schroeder](http://www.digitalprognosis.com) says:[28 April 2008 4:23 PM](#0247574cf5dd229b5f8192ed32b1bce11e26cc7d)

   Take a look at waf, besides requiring python it is much MUCH simpler to wrap your head around. It is also quite fast.

   http://code.google.com/p/waf/

2. [Tom Tromey](http://tromey.com/blog/) says:[1 May 2008 0:56 AM](#2399ad498d12f902a05971dbffd56b872882ea77)

   If you don't mind, could you report what you tried that did not work? You could email it to the list or just file it in the issues. I'll fix it at some point.

   Thanks, and thanks for trying Quagmire. FWIW I thought your review was very fair :-)

Comments are closed.
