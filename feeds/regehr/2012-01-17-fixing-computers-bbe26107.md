---
title: Fixing Computers
url: https://blog.regehr.org/archives/665
published: "2012-01-17T00:17:38Z"
feed: regehr
guid: http://blog.regehr.org/?p=665
---

# Fixing Computers

Pretty often, a friend or relative asks if I can fix something that’s wrong with their computer. Because I’m very irritable in some respects, my first impulse is always to say something like “Sure! And I bet a brain surgeon would love to put a band-aid on that ouchie, too.” Usually, I manage to resist. Over the years I’ve found that fixing random PC problems is a surprisingly entertaining form of puzzle solving. Also, when successful, it makes people really happy. This post contains a few things I’ve learned.

Most often the problem is easy to diagnose and also easy to fix, like a flaky fan, a full disk, not enough RAM, flaky RAM, a cable or I/O card is loose, or a peripheral has died.

A noisy fan should always be fixed. Even if the noise isn’t especially irritating, it probably stems from a fan that’s about to fail because it’s rubbing against something or its bearings are wearing out. Failed fans lead to nasty secondary consequences, so it’s best to just nip problems in the bud. Identifying the noisy fan is super easy: use a pencil or similar to stop each externally-reachable fan, one at a time. If this fails, crack open the case and repeat for each internal fan.

Some problems are easiest to diagnose by swapping out parts. Unfortunately, most people don’t keep a reserve of spare parts, but even so, common stuff like keyboard and mouse, monitor and video card, DVD drive, etc. are not hard to deal with. When in doubt, throwing a bit of money at the problem often works.

Some problems are *very* hard to diagnose. One of the worst I’ve seen stemmed from a slightly flaky DIMM and also a slightly flaky CDROM drive. Swapping out either part made the problem better, but did not make it go away.

Some fraction of the things that go wrong with Windows machines — malware, DLL hell, disk corruption, and similar — simply cannot be fixed. Luckily, these problems often only become serious fairly late in a computer’s useful operating life. Thus, people don’t mind too much being told they’re stuck with the problem unless they feel like re-installing the OS. Other Windows problems can be fixed by a mixture of Googling, registry editing, judicious uninstallation of crapware, etc.

It’s best to err on the side of caution, especially for old machines that are held together using duct tape and baling wire. Not too long ago I permanently killed my wife’s primary work machine either by defragmenting the hard disk or by uninstalling a few programs that I suspected were slowing down the boot process. Neither of these actions should have stopped the machine from booting — but one of them did, possibly by exposing some latent HDD corruption. Of course the dead machine was my fault and I had to dig myself out by building her a new, fast machine.

Finally, people are generally miserably poor about backups and computer problems present an excellent excuse to lecture them on backup discipline. These days it couldn’t be easier to perform proper backups, even if it’s just copying the “my documents” folder to a USB dongle or Dropbox disk every day or two.
