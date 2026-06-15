---
title: Digispark and Debian
url: https://nullprogram.com/blog/2014/05/14/
published: "2014-05-14T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2014/05/14/
---

# Digispark and Debian

## [Digispark and Debian](/blog/2014/05/14/)

May 14, 2014

nullprogram.com/blog/2014/05/14/

Following [Brian’s](http://www.50ply.com/) lead, I recently picked up a couple of [Digispark USB development boards](http://digistump.com/products/1). It’s a cheap, tiny, Arduino-like microcontroller. There are a couple of interesting project ideas that I have in mind for these. It’s [been over 6
years](/blog/2008/02/04/) since I last hacked on a microcontroller.

![](/img/misc/digispark-small.jpg)

Unfortunately, support for the Digispark on Linux is spotty. Just as with any hardware project, the details are irreversibly messy. It can’t make use of the standard Arduino software for programming the board, so you have to download a customized toolchain. This download includes files that have the incorrect vendor ID, requiring a manual fix. Worse, [the fix listed in their documentation](http://digistump.com/wiki/digispark/tutorials/linuxtroubleshooting) is incomplete, at least for Debian and Debian-derived systems.

The main problem is that Linux will *not* automatically create a `/dev/ttyACM0` device like it normally does for Arduino devices. Instead it gets a long, hidden, unpredictable device name. The fix is to ask udev to give it a predictable name by appending the following to the first line in the provided udev rules file ( `49-micronucleus.rules`),

```
SYMLINK+="ttyACM%n"

```

The whole uncommented portion of the rules file should look like this:

- [49-micronucleus.rules](http://pastebin.com/2XxmvEaS) (pastebin since it’s a long line)

The `==` is a conditional operator, indicating that the rule only applies when the condition is met. The `:=` and `+=` are assignment operators, evaluated when all of the conditions are met. The `SYMLINK` part tells udev put a softlink to the device in `/dev` under a predictable name.

- [meatspace](/tags/meatspace/)
- [tutorial](/tags/tutorial/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Digispark%20and%20Debian) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Digispark+and+Debian).

« [An Emacs Foreign Function Interface](/blog/2014/04/26/)

» [Emacs Lisp Buffer Passing Style](/blog/2014/05/27/)
