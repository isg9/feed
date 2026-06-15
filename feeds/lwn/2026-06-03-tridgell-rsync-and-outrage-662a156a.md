---
title: 'Tridgell: rsync and outrage'
url: https://lwn.net/Articles/1076040/
published: "2026-06-03T13:00:46Z"
feed: lwn
guid: https://lwn.net/Articles/1076040/
---

# Tridgell: rsync and outrage

Andrew Tridgell has written a [blog
post](https://medium.com/@tridge60/rsync-and-outrage-d9849599e5a0) responding to complaints that he has begun using LLM tools in his work maintaining [rsync](https://rsync.samba.org/):

> Like many developers of open source packages I've been hit by a flood of security reports lately in my role as the rsync maintainer. Many of those reports are AI generated (not all though, there are some notable ones with very careful and high quality manual analysis).
>
> As this flood started to get more intense I realised I needed to raise the defences on rsync a lot — we needed much more thorough test suites, code coverage analysis, CI testing on a lot more platforms, deliberate and thorough scanning for possible security issues (so I find at least some of them before other people!) and the addition of a whole lot of defence-in-depth hardening techniques.
>
> \[...\] Now to the future, because we're not done yet by a long shot. The security reports keep rolling in. I'm working on a bunch of CVEs right now. Luckily I've been joined by some other very good developers with great systems development skills and security knowledge. Some of these people came to my attention partly because of all the rage happening at the moment, so I get some rage storm clouds have silver linings. Watch out for some credits for some great new rsync developers in the next release.
