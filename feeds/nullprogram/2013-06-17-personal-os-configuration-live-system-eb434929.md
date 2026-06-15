---
title: Personal OS Configuration Live System
url: https://nullprogram.com/blog/2013/06/17/
published: "2013-06-17T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/06/17/
---

# Personal OS Configuration Live System

## [Personal OS Configuration Live System](/blog/2013/06/17/)

June 17, 2013

nullprogram.com/blog/2013/06/17/

I don’t know what to title or name this thing, so bear with me.

Two years ago I started [versioning my Emacs configuration](/blog/2011/10/19/). One year ago I started [versioning the rest of my dotfiles](/blog/2012/06/23/) the same way. This has composed beautifully with Debian, which truly is [*the* universal operating system](http://www.debian.org/). To create my comfortable development environment from scratch on a new (or used) computer, all I need to do is [install a bare-bones Debian system](http://www.debian.org/CD/netinst/), then direct it to automatically install a short list of my preferred software (apt-get), and finally clone these two repositories into place. Given a decent Internet connection, the whole process takes under an hour to go from blank hard drive to highly-productive computer system.

In fact, this whole process is so straightforward that it can be automated using an amazing tool called [live-build](http://live.debian.net/)! Taking the next step in versioning and automation, I wrote a live-build build that creates a live system with my personal configuration baked in.

- [https://github.com/skeeto/live-dev-env](https://github.com/skeeto/live-dev-env)

**A link to the latest ISO build can be found in the above link**. To try it out, burn it to a CD, write it onto a flash drive (it’s a hybrid ISO), or just fire it up in your favorite virtual machine. You will be booted directly into *very nearly* my exact configuration, down to the same random wallpaper selection. It’s extremely minimal and will look like this.

![](/img/screenshot/live-skeeto.jpg)

I don’t know for sure yet if this will be useful to anyone except me. On occasions where I need to make quick use of some arbitrary computer, or maybe just for system rescue, having this will be incredibly handy. [Knoppix](http://www.knopper.net/knoppix/index-en.html) is nice, but working without my own configuration can be discouraging; it’s so slow in comparison.

It’s also a chance for others to glimpse at my workflow without any commitment (i.e. potential dotfile clobbering). I like to study other developers’ workflows, stealing their ideas for my own, so I want to make mine easy to study. I think I’m doing some innovative things with a [hacked-together pseudo-tiling window manager](https://github.com/skeeto/dotfiles#openbox), my Firefox configuration, and my Emacs configuration. At least two of my co-workers’ Emacs configurations are forked from mine (you know who you are!). From a selfish perspective, the more people using workflows like mine, the better these workflows will be supported by the community at large!

I had said that it’s “very nearly” my exact configuration. At the time of this writing, what’s missing is [Quicklisp](http://www.quicklisp.org/) and [Leiningen](https://github.com/technomancy/leiningen), since these aren’t available as fully-functioning Debian packages at the moment. I’ll work them in eventually. The build is non-incremental and takes about an hour right now, so adding these little extras by trial-and-error will take some time.

[Pentadactyl](http://5digits.org/pentadactyl/) really shines here, because it allows me to completely configure Firefox/Iceweasel from a version-friendly text file. Except for some user scripts (still figuring out how to install those at build time), the browser in that image is identical to what I’m using right now. Even though [V8 is the king of performance](/blog/2013/02/25/), Firefox still wins hands-down for power users due to its superior configurability.

However, this Firefox configuration still has an annoyance. Several of the browser add-ons that I pre-install always pop up their first-run welcome messages. These messages flip a setting in Firefox’s registry so they don’t run again, a setting that is lost when the system shuts down. I can toggle these settings from the Pentadactyl configuration file, but not early enough. By the time Pentadactyl gets to applying these settings it’s too late. The messages, including Pentadactyl’s, are already queued to be shown. I don’t think it’s possible to completely fix this, even if Pentadactyl is fixed.

Anything not already installed is readily installable through apt-get. Right now I’m considering the feasibility of some sort of lazy-install system based on command-not-found. Debian has a package called command-not-found which intercepts the shell’s error handler when an issued command is not found. Instead of just giving the normal warning, it prints out what package needs to be installed in order to provide the command. What I think would be really neat is if the needed package is automatically installed, then the requested command is then re-run, all without returning control to the shell in the interim. It would be a lot like Emacs autoloads. As long as I have an Internet connection, most of Debian’s packages would be *virtually* installed on my live system as far as I’m concerned. The initial run of any program just takes a little longer.

I’ll continue to tweak this image over time, not only as I figure out how to make things work in Debian live-build, but also as my preferences and workflows evolve. Adjusting my configuration to work on a live-system has been enlightening, revealing all sorts of little manual things that I hadn’t yet automated. Perhaps someday this build will replace traditional operating system installations for me, at least for productive work. I could do all of my work from a portable read-only live system with a bit of short-lived (i.e. local cache) user-data persistence stored on a separate writable medium.

- [debian](/tags/debian/)
- [emacs](/tags/emacs/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Personal%20OS%20Configuration%20Live%20System) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Personal+OS+Configuration+Live+System).

« [An HTML5 Canvas Design Pattern](/blog/2013/06/16/)

» [Liquid Simulation in WebGL](/blog/2013/06/26/)
