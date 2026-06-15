---
title: canonical source — wingolog
url: https://wingolog.org/archives/2005/01/08/canonical-source
published: "2005-01-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/01/08/canonical-source
---

# canonical source — wingolog

## [canonical source](/archives/2005/01/08/canonical-source)

9 January 2005 0:14 AM

- [america](/tags/america)
- [hacks](/tags/hacks)

**travels**

I'm a tourist in my own country now. Well, not really -- I'm mostly visiting friends rather than places -- but I still haven't slept in the same place for a week straight. Here are a few photos:

![](//wingolog.org/wp-content/blue-house.jpg)*Leif's lovely blue house*

![](//wingolog.org/wp-content/krew.jpg)*Me, Brynn, and Leif on the blue ridge parkway*

![](//wingolog.org/wp-content/dirt-creek.jpg)*Dirt Creek Band -- Dad's on banjo*

![](//wingolog.org/wp-content/durham-amtrak-station.jpg)*The Amtrak station in Durham*

![](//wingolog.org/wp-content/old-homeplace.jpg)*Mountain living*

**computers**

I got a new hard drive for my old laptop and installed [Ubuntu](http://ubuntulinux.org/) on it. The brown is surprisingly nice, and the Applications / Places / System menus seem well-thought out. I went ahead and upgraded to Hoary. No problems here. To me, it's just debian with better gnome support.

I'm trying out [bazaar](http://bazaar.canonical.com/) as well. Arch is an interesting phenomenon -- you have a number of smart, strong-minded folks exploring the revision control problem space, with tensions from the different directions (GNU, Tom Lord, Bazaar, PyArch..), but united in their fundamental approach. I find it fascinating that these competing efforts can share a mailing list and an irc channel.

Bazaar does seem to provide some usability improvements over tla. I'm happy with it for now, and since it interoperates with tla-format archives, I imagine I'll use it in the future as well.

I started using hard-linked [revision libraries](http://regexps.srparish.net/www/tutorial/html/revision-libraries.html) with baz/tla (both [greedy and sparse](http://regexps.srparish.net/www/tutorial/html/advanced-revision-libraries.html)). I ran into some problems at first, because I had emacs set to not make backup files. Because the revision library and the working copy point to the same inode, editing the working copy affects the "pristine" source as well.

The solution to the problem is to turn on backup files, of course. Because I don't like lots of ~ files cluttering up my work space, I set the backups to go to `~/.backups/` with this `.emacs` snippet:

```
(custom-set-variables
 ; just copy the two lines below into the custom-set-variables block
 '(backup-directory-alist (quote ((".*" . "~/.backups"))))
 '(make-backup-files t)
)

```

This means that before emacs writes your file, it moves the old copy to ~/.backups. Thus the original inode is not affected, and the revision library keeps its integrity. If you mess up somehow, arch will know anyway -- you just have to remove the revision from your library. The next time you need it, arch will make a fresh copy.

## related articles

- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [digital interfaces](/archives/2009/11/07/digital-interfaces)
- [read what i wrote on my shirt](/archives/2009/10/08/read-what-i-wrote-on-my-shirt)
- [random things](/archives/2007/11/08/random-things)
- [bread crumbs](/archives/2007/06/08/bread-crumbs)

Comments are closed.
