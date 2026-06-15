---
title: failure to fail — wingolog
url: https://wingolog.org/archives/2003/12/03/advogato-25
published: "2003-12-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2003/12/03/advogato-25
---

# failure to fail — wingolog

## [failure to fail](/archives/2003/12/03/advogato-25)

3 December 2003 6:37 AM

- [random](/tags/random)
- [computers](/tags/computers)

**revision control**

While working on guile-gnome, another contributor started using Arch for some serious branching work. He seemed to be getting a lot of work done, and so I fetched arch and its tutorial. But I didn't know anything about subversion, so I started to check it out. There's an interesting [analysis of the situation with subversion](http://lists.fifthvision.net/pipermail/arch-users/2003-February/025283.html) by the author of arch, Tom Lord. The part that is really interesting to me comes at the end, when he discusses the svn developers:

> What's really disappointed me most, though is that, while I do perceive them as smart and ambitious, they don't seem terribly open-minded about stepping back to review their project for deep structural mistakes that need fixing. My sense is that most of them are pretty young and several have been associated with some successful projects (like Apache and CVS) -- good, young programmers, since they tend to be capable of so much more than their average peers, often fall into a pit of overconfidence which is hard to recognize from the inside until you've experienced a few disasters. The situation is made worse since there's so little effective mentoring in the industry from old-salts who are good at making a religion of the K.I.S.S. principle and making fun of the wealth of bloated, crappy, yet slow-to-fail stall-ware projects that dominate so much of the landscape. If you ask me, explosive growth during the dot-com bubble really blunted the technology edges of the free software movment and our industry generally. It left us collectively struggling to do things the hard way, svn being just one small example.

Interesting to me, mainly because I was [humbled](http://mail.gnu.org/archive/html/guile-devel/2003-11/msg00036.html) by Tom with regards to unicode in scheme. And because I've continued along wrong tracks before, just because that was where I was going. "Failure to Fail", he calls it: really just failing to fail at the proper time, which is a tough thing to swallow for many of us, myself included.

**components**

I have to finish wrapping some parts of the Gnome stack for guile-gnome. In doing so, I got into looking at the nasty parts of libbonoboui docs and my head started to spin -- why oh why would someone want to pain themselves with CORBA? There has to be a better way. For now, I just go the way of making libs out of apps that have reusable components. Not ideal, but not painful either.

## related articles

- [Roundup](/archives/2005/10/27/roundup)
- [speculations](/archives/2002/10/05/advogato-15)
- [mastery](/archives/2002/08/11/advogato-13)
- [bigass truck](/archives/2002/06/14/advogato-2)
- [come on feet](/archives/2006/05/13/come-on-feet)
- [american dispatches](/archives/2006/03/04/american-dispatches)

Comments are closed.
