---
title: Distributed Issue Tracking
url: https://nullprogram.com/blog/2009/02/14/
published: "2009-02-14T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/02/14/
---

# Distributed Issue Tracking

## [Distributed Issue Tracking](/blog/2009/02/14/)

February 14, 2009

nullprogram.com/blog/2009/02/14/

In a [previous post](/blog/2009/02/12#git-is-better) I discussed decentralized version control systems, Git in particular. Because decentralized version control is becoming so popular, we now have an exciting new area of development: distributed issue tracking.

Decentralized issue tracking seems to have popped into existance in the last year or so. A number of projects have appeared (cil, ticgit, ditz, to name some), but the one that really stands out for me is [ditz](http://ditz.rubyforge.org/). Keep an eye on that one. It's fairly active and mostly usable.

Decentralized trackers generally work by storing the issue tracking database within the repository itself. One possibility is to have it sit in its own branch, which I think is the Wrong Way. A second possibility is to have it sit right next to the code in its own directory. Yet another possibility is to put the issue tracker in its own repository. Git could even include this repository as a submodule (this is a lot like the Wrong Way, though).

First of all, everyone gets their own copy of the issue tracker database and its history. Second of all, it *has* history. It's tracked the same way the code is. And, in the second case usage, one of the coolest advantages is that issues follow the code very closely.

When a branch is created, it takes its own copy of the issue tracking database with it. If a bug is fixed in the main branch, the issue tracking database in the main branch is updated. The bug will remain in the side branch and the issue will still be open in the side branch reflecting this. If a merge occurs later, the issue tracking database also gets merged automatically. I think that's damn cool.

There are some issues that still need to be hammered out. How does a non-developer enter a ticker? They would need to work the version control system to do this, then be able to share that change. That's a pretty large barrier.

Perhaps a web interface could be set up for setting up issues. Developers could then cherry-pick/pull the issues from that repository and push ticket updates back out.

Then there is overhead incurred by moving tickets around with code. How bad is this overhead? How can this be dealt with in the most transparent way? This all needs to be tested still.

Could the issue database get too big? People like to attach screenshots to issues. Having many screenshots would make the repository very big. How do we deal with this?

It's an exciting, new realm to explore.

- [git](/tags/git/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Distributed%20Issue%20Tracking) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Distributed+Issue+Tracking).

« [Git is Better](/blog/2009/02/12/)

» [Readline Wrap](/blog/2009/02/20/)
