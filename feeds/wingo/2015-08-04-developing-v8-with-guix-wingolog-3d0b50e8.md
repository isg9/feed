---
title: developing v8 with guix — wingolog
url: https://wingolog.org/archives/2015/08/04/developing-v8-with-guix
published: "2015-08-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/08/04/developing-v8-with-guix
---

# developing v8 with guix — wingolog

## [developing v8 with guix](/archives/2015/08/04/developing-v8-with-guix)

4 August 2015 4:23 PM

- [v8](/tags/v8)
- [guix](/tags/guix)
- [nix](/tags/nix)
- [functional package management](/tags/functional%20package%20management)
- [guile](/tags/guile)
- [igalia](/tags/igalia)

**a guided descent into hell**

It all started off so simply. My primary development machine is a desktop computer that I never turn off. I suspend it when I leave work, and then resume it when I come back. It's always where I left it, as it should be.

I rarely update this machine because it works well enough for me, and anyway my focus isn't the machine, it's the things I do on it. Mostly I work on V8. The setup is so boring that I certainly didn't imagine myself writing an article about it today, but circumstances have forced my hand.

This machine runs Debian. It used to run the testing distribution, but somehow in the past I needed something that wasn't in testing so it runs unstable. I've been using Debian for some 16 years now, though not continuously, so although running unstable can be risky, usually it isn't, and I've unborked it enough times that I felt pretty comfortable.

Perhaps you see where this is going!

I went to install something, I can't even remember what it was now, and the downloads failed because I hadn't updated in a while. So I update, install the thing, and all is well. Except my instant messaging isn't working any more because there are a few moving parts (empathy / telepathy / mission control / gabble / dbus / whatwhat), and the install must have pulled in something that broke one of them. No biggie, this happens. Might as well go ahead and update the rest of the system while I'm at it and get a reboot to make sure I'm not running old software.

Most Debian users know that you probably shouldn't do a `dist-upgrade` from an old system -- you `upgrade` and then you `dist-upgrade`. Or perhaps this isn't even true, it's tribal lore to avoid getting eaten by the wild beasts of bork that roam around the village walls at night. Anyway that's what I did -- an `upgrade`, let it chunk for a while, then a `dist-upgrade`, check the list to make sure it didn't decide to remove one of my kidneys to satisfy the priorities of the bearded demon that lives inside `apt-get`, OK, let it go, all is well, reboot. Swell.

Or not! The computer restarts to a blank screen. Ha ha ha you have been bitten by a bork-beast! Switch to a terminal and try to see what's going on with GDM. It's gone! Ha ha ha! Your organs are being masticated as we speak! How does that feel! Try to figure out which package is causing it, happily with another computer that actually works. Surely this will be fixed in some update coming soon. Oh [it's something that's going to take a few weeks!!!!](https://lists.debian.org/debian-user/2015/08/msg00044.html) Ninth level, end of the line, all passengers off!

**my gods**

I know how we got here, I love Debian, but it is just unacceptable and revolting that software development in 2015 is exposed to an upgrade process which (1) can break your system (2) by default and (3) can't be rolled back. The last one is the killer: who would design software this way? If you make a system like this in 2015 I'd say you're committing malpractice.

Well yesterday I resolved that this would be the last time this happens to me. Of course I could just develop in a virtual machine, and save and restore around upgrades, but that's kinda trash. Or I could use btrfs and be able to rewind changes to the file system, but then it would rewind everything, not just the system state.

Fortunately there is a better option in the form of functional package managers, like [Nix](http://nixos.org/) and [Guix](http://gnu.org/s/guix). Instead of upgrading your system by mutating `/usr`, Nix and Guix store all files in a content-addressed store ( `/nix/store` and `/gnu/store`, respectively). A user accesses the store via a "profile", which is a forest of symlinks into the store.

For example, on my machine with a NixOS system installation, I have:

```
$ which ls
/run/current-system/sw/bin/ls

$ ls -l /run/current-system/sw/bin/ls
lrwxrwxrwx 1 root nixbld 65 Jan  1  1970
  /run/current-system/sw/bin/ls ->
    /nix/store/wc472nw0kyw0iwgl6352ii5czxd97js2-coreutils-8.23/bin/ls

$ ldd /nix/store/wc472nw0kyw0iwgl6352ii5czxd97js2-coreutils-8.23/bin/ls
  linux-vdso.so.1 (0x00007fff5d3c4000)
  libacl.so.1 => /nix/store/c2p56z920h4mxw12pjw053sqfhhh0l0y-acl-2.2.52/lib/libacl.so.1 (0x00007fce99d5d000)
  libc.so.6 => /nix/store/la5imi1602jxhpds9675n2n2d0683lbq-glibc-2.20/lib/libc.so.6 (0x00007fce999c0000)
  libattr.so.1 => /nix/store/jd3gggw5bs3a6sbjnwhjapcqr8g78f5c-attr-2.4.47/lib/libattr.so.1 (0x00007fce997bc000)
  /nix/store/la5imi1602jxhpds9675n2n2d0683lbq-glibc-2.20/lib/ld-linux-x86-64.so.2 (0x00007fce99f65000)

```

Content-addressed linkage means that files in the store are never mutated: they will *never* be overwritten by a software upgrade. Never. Never will I again gaze in horror at the frozen beardcicles of a Debian system in the throes of "oops I just deleted all your programs, like that time a few months ago, wasn't that cool, it's really cold down here, how do you like my frozen facial tresses and also the horns".

At the same time, I don't have to give up upgrades. Paradoxically, immutable software facilitates change and gives me the freedom to upgrade my system without anxiety and lost work.

**nix and guix**

So, there's Nix and there's Guix. Both are great. I'll get to comparing them, but first a digression on the ways they can be installed.

Both Nix and Guix can be installed either as the operating system of your computer, or just as a user-space package manager. I would actually recommend to people to start with the latter way of working, and move on to the OS if you feel comfortable. The fundamental observation here is that because `/nix/store` doesn't depend on or conflict with `/usr`, you can run Nix or Guix as a user on a (e.g.) Debian system with no problems. You can have a forest of symlinks in `~/.guix-profile/bin` that links to nifty things you've installed in the store and that's cool, you don't have to tell Debian.

**and now look at me**

In my case I wanted to also have the system managed by Nix or Guix. GuixSD, the name of the Guix OS install, isn't appropriate for me yet because it doesn't do GNOME. I am used to GNOME and don't care to change, so I installed NixOS instead. It works fine. There have been some irritations -- for example it just took me 30 minutes to figure out how to install `dict`, with a local wordnet dictionary server -- but mostly it has the packages I need. Again, I don't recommend starting with the OS install though.

GuixSD, the OS installation of Guix, is a bit harder even than NixOS. It has fewer packages, though what it does have tends to be more up-to-date than Nix. There are two big things about GuixSD though. One is that it aims to be fully free, including avoiding non-free firmware. Because they build deterministic build products from source, Nix and Guix can offer completely reproducible builds, which is swell for software reliability. Many reliability people also care a lot about software freedom and although Nix does support software freedom very well, it also includes options to turn on the Flash plugin, for example, and of course includes the Linux kernel with all of the firmware. Well GuixSD eschews non-free firmware, and uses the Linux-Libre kernel. For myself I have a local build on another machine that uses the stock Linux kernel with firmware for my Intel wireless device, and I was really discouraged from even sharing the existence of this hack. I guess it makes sense, it takes a world to make software freedom, but that particular part is not my fight.

The other thing about Guix is that it's really GNU-focused. This is great but also affects the product in some negative ways. They use "dmd" as an init system, for example, which is kinda like systemd but not. One consequence of this is that GuixSD doesn't have an implementation of the `org.freedesktop.login1` seat management interface, which these days is implemented by part of systemd, which in turn precludes a bunch of other things GNOME-related. At one point I started working on a [fork of systemd that pulled logind out to a separate project](https://github.com/andywingo/elogind), which makes sense to me for distros that want seat management but not systemd, but TBH I have no horse in the systemd race and in fact systemd works well for me. But, a system with elogind would also work well for me. Anyway, the upshot is that unless you care a lot about the distro itself or are willing to adapt to e.g. Xfce or Xmonad or something, NixOS is a more pragmatic choice.

**i'm on a horse**

I actually like Guix's tools better than Nix's, and not just because they are written in Guile. Guix also has all the tools I need for software development, so I prefer it and ended up installing it as a user-space package manager on this NixOS system. Sounds bizarre but it actually works pretty well.

So, the point of this article is to be a little guide of how to build V8 with Guix. Here we go!

**up and running with guix**

First, [check the manual](https://www.gnu.org/software/guix/manual/html_node/index.html). It's great and well-written and answers many questions and in fact includes all of this.

Now, I assume you're on an x86-64 Linux system, so we're going to use the awesome binary installation mechanism. Check it out: because everything in `/gnu/store` is linked directly to each other, all you have to do is to copy a reified `/gnu/store` onto a working system, then copy a sqlite thing into `/var`, and you've installed Guix. Sweet, eh? And actually you can take a running system and clone it onto other systems in that way, and Guix even provides a [tool to generate such a tarball for you](https://www.gnu.org/software/guix/manual/html_node/Binary-Installation.html#Binary-Installation). Neat stuff.

```
cd /tmp
wget ftp://alpha.gnu.org/gnu/guix/guix-binary-0.8.3.x86_64-linux.tar.xz
tar xf guix-binary-0.8.3.x86_64-linux.tar.xz
mv var/guix /var/ && mv gnu /

```

This Guix installation has a built-in profile for the root user, so let's go ahead and add a link from `~root` to the store.

```
ln -sf /var/guix/profiles/per-user/root/guix-profile \
       ~root/.guix-profile

```

Since we're root, we can add the `bin/` part of the Guix profile to our environment.

```
export PATH="$HOME/.guix-profile/bin:$HOME/.guix-profile/sbin:$PATH"

```

Perhaps we add that line to our `~root/.bash_profile`. Anyway, now we have Guix. Or rather, we almost have Guix -- we need to start the daemon that actually manages the store. Create some users:

```
groupadd --system guixbuild

for i in `seq -w 1 10`; do
  useradd -g guixbuild -G guixbuild           \
          -d /var/empty -s `which nologin`    \
          -c "Guix build user $i" --system    \
          guixbuilder$i;
done

```

And now run the daemon:

```
guix-daemon --build-users-group=guixbuild

```

If your host distro uses systemd, there's a unit that you can drop into the systemd folder. See the manual.

A few more things. One, usually when you go to install something, you'll want to fetch a pre-built copy of that software if it's available. Although Guix is fundamentally a build-from-source distro, Guix also runs a continuous builder service to make sure that binaries are available, if you trust the machine building the binaries of course. To do that, we tell the daemon to trust `hydra.gnu.org`:

```
guix archive --authorize < ~root/.guix-profile/share/guix/hydra.gnu.org.pub

```

**as a user**

OK now we have Guix installed. Running Guix commands will install things into the store as needed, and populate the forest of symlinks in the current user's `$HOME/.guix-profile`. So probably what you want to do is to run, as your user:

```
/var/guix/profiles/per-user/root/guix-profile/bin/guix \
  package --install guix

```

This will make Guix available in your own user's profile. From here you can begin to install software; for example, if you run

```
guix package --install emacs

```

You'll then have an emacs in `~/.guix-profile/bin/emacs` which you can run. Pretty cool stuff.

**back on the horse**

So what does it mean for software development? Well, when I develop software, I usually want to know exactly what the inputs are, and to not have inputs to the build process that I don't control, and not have my build depend on unrelated software upgrades on my system. That's what Guix provides for me. For example, when I develop V8, I just need a few things. In fact I need these things:

```
;; Save as ~/src/profiles/v8.scm
(use-package-modules gcc llvm base python version-control less ccache)

(packages->manifest
 (list clang
       coreutils
       diffutils
       findutils
       tar
       patch
       sed
       grep
       binutils
       glibc
       glibc-locales
       which
       gnu-make
       python-2
       git
       less
       libstdc++-4.9
       gcc-4.9
       (list gcc-4.9 "lib")
       ccache))

```

This set of Guix packages is what it took for me to set up a V8 development environment. I can make a development environment containing only these packages and no others by saving the above file as `v8.scm` and then sourcing this script:

```
~/.guix-profile/bin/guix package -p ~/src/profiles/v8 -m ~/src/profiles/v8.scm
eval `~/.guix-profile/bin/guix package -p ~/src/profiles/v8 --search-paths`
export GYP_DEFINES='linux_use_bundled_gold=0 linux_use_gold_flags=0 linux_use_bundled_binutils=0'
export CXX='ccache clang++'
export CC='ccache clang'
export LD_LIBRARY_PATH=$HOME/src/profiles/v8/lib

```

Let's take this one line at a time. The first line takes my manifest -- the set of packages that collectively form my build environment -- and arranges to populate a symlink forest at `~/src/profiles/v8`.

```
$ ls -l ~/src/profiles/v8/
total 44
dr-xr-xr-x  2 root guixbuild  4096 Jan  1  1970 bin
dr-xr-xr-x  2 root guixbuild  4096 Jan  1  1970 etc
dr-xr-xr-x  4 root guixbuild  4096 Jan  1  1970 include
dr-xr-xr-x  2 root guixbuild 12288 Jan  1  1970 lib
dr-xr-xr-x  2 root guixbuild  4096 Jan  1  1970 libexec
-r--r--r--  2 root guixbuild  4138 Jan  1  1970 manifest
lrwxrwxrwx 12 root guixbuild    59 Jan  1  1970 sbin -> /gnu/store/1g78hxc8vn7q7x9wq3iswxqd8lbpfnwj-glibc-2.21/sbin
dr-xr-xr-x  6 root guixbuild  4096 Jan  1  1970 share
lrwxrwxrwx 12 root guixbuild    58 Jan  1  1970 var -> /gnu/store/1g78hxc8vn7q7x9wq3iswxqd8lbpfnwj-glibc-2.21/var
lrwxrwxrwx 12 root guixbuild    82 Jan  1  1970 x86_64-unknown-linux-gnu -> /gnu/store/wq6q6ahqs9rr0chp97h461yj8w9ympvm-binutils-2.25/x86_64-unknown-linux-gnu

```

So that's totally scrolling off the right for you, that's the thing about Nix and Guix names. What it means is that I have a tree of software, and most directories contain a union of links from various packages. It so happens that `sbin` though just has links from `glibc`, so it links directly into the store. Anyway. The next line in my `v8.sh` arranges to point my shell into that environment.

```
$ guix package -p ~/src/profiles/v8 --search-paths
export PATH="/home/wingo/src/profiles/v8/bin:/home/wingo/src/profiles/v8/sbin"
export CPATH="/home/wingo/src/profiles/v8/include"
export LIBRARY_PATH="/home/wingo/src/profiles/v8/lib"
export LOCPATH="/home/wingo/src/profiles/v8/lib/locale"
export PYTHONPATH="/home/wingo/src/profiles/v8/lib/python2.7/site-packages"

```

Having sourced this into my environment, my shell's `ls` for example now points into my new profile:

```
$ which ls
/home/wingo/src/profiles/v8/bin/ls

```

Neat. Next we have some V8 defines. On x86\_64 on Linux, v8 wants to use some binutils things that it bundles itself, but oddly enough for months under Debian I was seeing spurious intermittent segfaults while linking with their bundled `gold` linker binary. I don't want to use their idea of what a linker is anyway, so I set some defines to make v8's build tool use Guix's linker. (Incidentally, figuring out what those defines were took spelunking through makefiles, to gyp files, to the source of gyp itself, to the source of the standard `shlex` Python module to figure out what delimiters `shlex.split` actually splits on... yaaarrggh!)

Then some defines to use ccache, then a strange thing: what's up with that `LD_LIBRARY_PATH`?

Well. I'm not sure. However the normal thing for dynamic linking under Linux is that you end up with binaries that are just linked against e.g. `libc.so.6`, whereever the system will find `libc.so.6`. That's not what we want in Guix -- we want to link against a *specific* version of every dependency, not just any old version. Guix's builders normally do this when building software for Guix, but somehow in this case I haven't managed to make that happen, so the binaries that are built as part of the build process can end up not specifying the path of the libraries they are linked to. I don't know whether this is an issue with v8's build system, that it doesn't want to work well with Nix / Guix, or if it's something else. Anyway I hack around it by assuming that whatever's in my artisanally assembled symlink forest ("profile") is the right thing, so I set it as the search path for the dynamic linker. Suggestions welcome here.

And from here... well it just works! I've gained the ability to precisely specify a reproducible build environment for the software I am working on, which is entirely separated from the set of software that I have installed on my system, which I can reproduce precisely with a script, and yet which is still part of my system -- I'm not isolated from it by container or VM boundaries (though I can be; see [NixOps](http://nixos.org/nixops/) for more in that direction).

OK I lied a little bit. I had to apply this patch to V8:

```
$ git diff
diff --git a/build/standalone.gypi b/build/standalone.gypi
index 2bdd39d..941b9d7 100644
--- a/build/standalone.gypi
+++ b/build/standalone.gypi
@@ -98,7 +98,7 @@
         ['OS=="win"', {
           'gomadir': 'c:\\goma\\goma-win',
         }, {
-          'gomadir': 'See?  Because my system is NixOS, there is no /bin/echo.  It does helpfully install a /usr/bin/env though, which other shell invocations in this build script use, so I use that instead.  I mention this as an example of what works and what workarounds there are.dpkg --purgatorySo now I have NixOS as my OS, and I mostly use Guix for software development.  This is a new setup and we'll see how it works in practice.Installing NixOS on top of Debian was a bit irritating.  I ended up making a bootable USB installation image, then installing over to my Debian partition, happy in the idea that it wouldn't conflict with my system.  But in that I forgot about /etc and /var and all that.  So I copied /etc to /etc-debian, just as a backup, and NixOS appeared to install fine.  However it wouldn't boot, and that's because some systemd state from my old /etc which was still in place conflicted with... something?  In the end I redid the install, moving my old /usr, /etc and such directories to backup names and letting NixOS have control.  That worked fine.I have GuixSD on a laptop but I really don't recommend it right now -- not unless you have time and are willing to hack on it.  But that's OK, install NixOS and you'll be happy on the system side, and if you want Guix you can install it as a user.Comments and corrections welcome, and happy hacking!
```

## related articles

- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)

### 21 responses

1. Dale Smith says:[4 August 2015 5:51 PM](#b84674ff51edda8d4acfb87d71b3938f4d1b1034)

   Oh how I feel for you! After upgrading to Jessie my laptop kept going to sleep about every 30 seconds. Needed to disable sensing the lit switch. Very irritating trying to find a solution in 15 second bursts!

   I have to try guix or nix soon. Been using Debian since 1.0 (yes 1.\*0\*, I had one of those jump-the-gun CDs), but things are just getting intolerable.

2. [qznc](http://beza1e1.tuxen.de) says:[4 August 2015 9:11 PM](#6b75b46b646e9882a49874e2bb7246f97d4136df)

   \> Nix and Guix can offer completely reproducible builds

   Since I have read about Debians Reproducible Builds project \[0\], I always doubt it when anybody claims that. Did the Nix/Guix maintainers really go through all the hassles and achieved that?

   \[0\] https://wiki.debian.org/ReproducibleBuilds/About

3. Austin Seipp says:[4 August 2015 10:25 PM](#787d13fc931a35f4eb3f4990d80393055313bad9)

   Unfortunately, there is a bit of confusing terminology here regarding 'deterministic' or 'reproducible' builds in NixOS land (and I think it's our fault, we should clarify this in the manual or homepage or something). I outlined the basic ideas and what NixOS users often mean by 'reproducible' in my GSoC 2015 proposal, and where we are now:

   https://nixos.org/wiki/GSOC\_2015\_ideas\_list#Deterministic\_Builds

   In practice, most of the work for a fully bit-deterministic NixOS ISO is there. I unfortunately just haven't had time to fully test and integrate it into the NixOS tree, and we were denied a GSoC slot this year. Hopefully I'll find the time to finish it...

4. Johan Ouwerkerk says:[5 August 2015 1:35 AM](#d744a1d28e2c95626be1de082be6996b30517120)

   Well as soon as I saw you ran Debian unstable... I knew where this was headed. Still, nix is pretty cool stuff so I can't complain. ;)

   However you might want to know a few things:

   (1) Debian unstable is actually the dumping ground for things in a meta-state between "horribly broken stuff that doesn't build but instead brings down the four horseman, locusts, plague and packaged bobcats on your head" to "merely buggy things that need fixing". There's basically zero QA apart from some build time checks, you are the tester, etc. Usually things aren't too bad, but right now they are in transition to GCC 5. This means world+dog needs recompilation, and their deps too and it's a iterative process of ironing out the missing deps as and when they hit the 'unstable' repository... This has been helpfully illustrated as the "Apocalypse" with what looks like a picture of the detonation of the "Tsar Bomba".

   (2) Also if don't want to stick to the confines of the safety of 'stable', instead of using apt-get consider using aptitude. It's got a considerably smarter upgrade logic including, in particularly w.r.t. conflict resolution (simply deny its solutions to cycle through them and see which ones you like best). Also, there is safe-upgrade to limit whole upgrade to just the subset that can be safely upgraded. So breakage is a lot less likely that way.

   (3) Minor point: for the love of sanity, don't use useradd. Use the adduser tool instead. useradd is an excellent way to break your system without the use of a package manager because instead of trying to do the sane thing (nothing) when you mistype, useradd will happily press the big red button marked "detonate doomsday advice" at the merest provocation. Like hitting enter just one keystroke too soon. ;)

5. Johan Ouwerkerk says:[5 August 2015 1:35 AM](#388b9c534e303d124757947fef9cde86802b2080)

   Well as soon as I saw you ran Debian unstable... I knew where this was headed. Still, nix is pretty cool stuff so I can't complain. ;)

   However you might want to know a few things:

   (1) Debian unstable is actually the dumping ground for things in a meta-state between "horribly broken stuff that doesn't build but instead brings down the four horseman, locusts, plague and packaged bobcats on your head" to "merely buggy things that need fixing". There's basically zero QA apart from some build time checks, you are the tester, etc. Usually things aren't too bad, but right now they are in transition to GCC 5. This means world+dog needs recompilation, and their deps too and it's a iterative process of ironing out the missing deps as and when they hit the 'unstable' repository... This has been helpfully illustrated as the "Apocalypse" with what looks like a picture of the detonation of the "Tsar Bomba".

   (2) Also if don't want to stick to the confines of the safety of 'stable', instead of using apt-get consider using aptitude. It's got a considerably smarter upgrade logic including, in particularly w.r.t. conflict resolution (simply deny its solutions to cycle through them and see which ones you like best). Also, there is safe-upgrade to limit whole upgrade to just the subset that can be safely upgraded. So breakage is a lot less likely that way.

   (3) Minor point: for the love of sanity, don't use useradd. Use the adduser tool instead. useradd is an excellent way to break your system without the use of a package manager because instead of trying to do the sane thing (nothing) when you mistype, useradd will happily press the big red button marked "detonate doomsday advice" at the merest provocation. Like hitting enter just one keystroke too soon. ;)

6. Brandon says:[5 August 2015 4:40 AM](#cf4f139b8bc024ec87eb7ebbdc3af48947baa8c6)

   TBH, I think apt and rpm are an "overly mature" optimization. We have so much disk space now, so much ram now, that I don't think it really makes sense to try and save a few MB here and there with dynamic linking and risk breaking the entire system to update one package.

   I once went through an LFS build as an exercise, which taught me that what it takes to get a functional distribution entirely from upstream sources is nothing short of voodoo black magic. It's this version of this package, with this patch applied, plus that version of that package with THAT patch applied. Rinse and repeat 100,000 times until you have all the packages your distribution needs and have resolved \*all\* the conflicts between each of them. It's completely crazy. Sooner or later, you'll have an irreconcilable conflict between some set of packages you want and their respective dependencies.

   The web community that has embraced things like \`npm\` and \`pip\`, where each project installs its own SEPARATE dependencies. At first I thought this was stupid and wasteful, but after a couple of years of working with it, in high-pressure startup environments, it makes a hell of a lot more sense than apt. Especially in 2015 where we measure disk capacity in terabytes.

   My experience working for a VMWare and trying to ship binaries that depended on system libraries convinced me that it's just not a feasible approach. In the end, due to the set of distros, architectures, and images we had to support, it constrained us to painfully-old versions of GTK and glib. For the Workstation folks, it means a fiendishly complicated installation process for both the Workstation binaries and the tools package that gets installed on the guest. Also that a company like VMware has to statically link whatever they legally can, and find a way to install customized versions of things they can't. When you have to depend on a system package, pain is the result.

   I've had experiences with debian unstable similar to yours and I share your pain. But that's why I switched to Ubuntu in 2010, and never looked back. Unstable is untrustworthy, and Testing is already out of date. Hell, ubuntu releases every 6 months and IT's always out-of-date. So in 2011 I bought a mac, and used a variety of VMs for development. When I still worked on PiTiVi, I built the \*entire\* gnome stack inside jh-build every couple of days.

   I never dist-upgrade an ubuntu system. Instead, I get the next livecd and upgrade from that, or I do a \*clean install\* over the top of the old system. Way more reliable than dist-upgrade ever was.

   In any event, I stopped seeing debian as a usable end-user system a long time ago. It's a useful resource for people putting together end-user distros. But you can't depend on it. Malpractice is a good way to describe it.

7. Brandon says:[5 August 2015 5:23 AM](#d88e2f6580e42f9784f8c6d9b3a9e9e15546d54a)

   The other thing I want to mention is that "content-addressable storage" was anticipated by Gobo linux as far back as ought three (In the days before git made everyone comfortable with the notion of identifying things you care about with cryptographic hashes). I thought it was a neat way around this very problem. Of course, Gobo linux was one man's unique madness, and it was ignored or derided by the larger community for breaking with the FSHS.

8. Martin says:[5 August 2015 7:53 AM](#4dd5ca76dbca661adfd991cc747db80c20ba65b3)

   While I see a lot of potential in nix/guix, for the specific case, here is how I solve this: I run Debian stable (at the office) or testing (at home) on the desktop, but maintain some chroots (stable, testing, unstable) for developement. Works fine for me.

9. Martin says:[5 August 2015 7:58 AM](#22934f7f9043f146bc21da3b60b53e677c0c9fc7)

   Brandon, your supposition that we have "so much disk space now, so much ram now" does not hold for me. I'm working in the embedded computing field with Debian and while can do an "apt-get update" + "apt-get upgrade" on our devices, an "apt-get dist-upgrade" to the next stable release is impossible.

10. [François-René Rideau](http://fare.tunes.org/computing/) says:[5 August 2015 11:20 AM](#04b50a14f44116082d8eb290a4b089ca2bf73498)

    Congrats using NixOS and Guix. Love them, too.

    Regarding the LD\_LIBRARY\_PATH, I don't know how Guix works exactly, but NixOS has a program to fixup the rpath of elf binaries after they are built so they become statically linked to the correct dependencies.

11. Brandon says:[5 August 2015 5:55 PM](#8882d5b217f31e75cc3ab0cc334903a2def30797)

    @Martin: In that case, I doubt nix or guix is going to work for you either. But in any case, if you're working in embedded, you wouldn't dist-upgrade the device itself, but rather flash a new firmware image built form the next release. At least, that's how I'd do it.

12. Martin says:[5 August 2015 10:20 PM](#df038a37695647f6cfdba62fa42dfe57a268e99b)

    Brandon, if the embedded device is remote and connected via a slow modem, flashing is not possible, dist-upgrade is, as long as the number or size of updated packages is not too high.

13. Brandon says:[6 August 2015 4:01 AM](#fc2640580a7da1df9427c0c1a916d614f82ca589)

    Martin, what on earth (or in space) are you working on? You have an embedded device and the file system is actually writable? You're actually doing over-the-air dist upgrade via some kind of slow connection? I think it's risky.

    The way I see it, if you have enough writable storage to download all the new packages; plus the extra space required to unzip all the packages; plus the extra space to hold the output of all the install scripts; you've got enough space to hold a file system image where all those steps have already been done. Plus you can test the image on your actual hardware before you roll the update out and know with a much greater degree of certainty how the update will proceed in the field.

14. Johann says:[6 August 2015 2:58 PM](#0d2d7f57371994cab39d0709a1ccb1d06cee59a0)

    Guix and Nix look interesting, but I have yet to explore them myself. I haven't found a downside of using Debian stable combined with compiling a few newer applications and installing them into /usr/local with GNU Stow. Then again, I don't do much linux hacking any more... that's been replaced with dissertation writing.

15. Sorpigal says:[8 August 2015 1:02 PM](#8b9a1dbf9ae8adef27aa0e0f4c16d4163bcd29d1)

    Johann,

    Guix looks to me a lot like the logical successor to GNU Stow--on steroids. Inspired by this article I'm trying it out right now on my Debian sid box.

16. Kelly says:[9 August 2015 3:46 AM](#d23bcd8dba9df4b7d6034159a67c9f973cbbec40)

    You dist-upgraded into unstable?? (and, how does dist-upgrade even work considering how unstable works...) and without manually checking all the packages? (that is where aptitude's ncurses mode comes in). I mean... what did you expect?

    I'd agree it was nuts if you were dist-upgrading from stable to stable, but you gotta understand, all bets are off if you are talking testing/unstable. I love unstable, use it as my main system, but shit, you just gotta be careful!

17. [clacke](https://microca.st/clacke) says:[9 August 2015 9:22 PM](#8bce8269c3e3ec74a12d78ba9656c01d39798590)

    Sorpigal and Johann:

    stow on steroids is exactly how I view nix and guix. They even do that dir symlink optimization. :-)

    Making a guix recipe isn't that much work, and you save some effort the next upgrade.

18. [Ludovic Courtès](http://www.fdn.fr/~lcourtes) says:[18 August 2015 1:02 PM](#43a09b33e9982d32d57af88a17553937adf98373)

    A bit late in the game, but thanks for the review!

    For your software environment use case, don't miss the 'guix environment' command, which is probably easier to use than 'guix package -p' (and will soon be able to set up an isolated container, too.)

19. Evan Rowley says:[20 August 2015 6:26 PM](#7f15ab1d6aed03a07051e87937a35e6832402846)

    Thank you sir for walking through the intricacies of debain package management hell, Nix/Guix, and what it's like to use Guix on NixOS. Well written and a great read.

20. Alex Vong says:[19 October 2015 4:42 AM](#2ac43537367bb9ef50fff16aacef939dbbe2c975)

    I have also run into the trouble in unstable with gnome before. I have set up auto-login in gnome, but gnome won't let me login after an upgrade. Luckily, I have Debian stable co-installed. The problem get fixed in about a week after another upgrade. (I run \`apt-get dist-upgrade' in tty1.) It seems gnome sometimes messes up in unstable. If you want to be "stable" in unstable, you should use xfce. But I am using gnome anyway since it has a lot of lovely features.

21. [Steve Darts](http://www.stevelocke.co.uk) says:[11 July 2016 4:25 PM](#600d98e7461003567ae8db1764c789c0293dc9a3)

    Most Debian users know that you probably shouldn't do a dist-upgrade from an old system.

Comments are closed.
