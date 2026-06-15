---
title: metablog — wingolog
url: https://wingolog.org/archives/2008/04/23/metablog
published: "2008-04-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/23/metablog
---

# metablog — wingolog

## [metablog](/archives/2008/04/23/metablog)

23 April 2008 12:54 PM

- [meta](/tags/meta)
- [tekuti](/tags/tekuti)
- [scheme](/tags/scheme)

Well, it's been a little while since I wrote about [tekuti](//wingolog.org/software/tekuti/), my Scheme-powered, git-backed blog software. Time for an update.

**features**

I added a global [tag cloud](//wingolog.org/tags/), in addition to the abridged cloud on the main page. Clicking on a tag takes you to a [time-ordered list of posts having that tag](//wingolog.org/tags/scheme), with a cloud of related tags. The post list could use some improvement, mostly because my titles are nonsensical.

I also implemented full-text search, which uses `git grep` under the hood. Amusing.

Also amusing is the "related posts" list, which shows up individual post pages. It's calculated as the set of posts which share the most tags with the post in question. The indexes that are automatically rebuilt when the master ref changes makes this a relatively cheap set to compute.

Also also: some artificial intelligence anti spam foo. Ha!

This stuff is actually fun to hack on, and is self-contained -- I'm almost spending more time writing about the features than I did implementing them.

**documentation**

I fleshed out [tekuti's web page](//wingolog.org/software/tekuti/) today, giving reasonably detailed install, deployment, and hacking instructions. It even includes a description on how to migrate from wordpress. I'd appreciate any comments that folks might have, probably better on this post than via email.

Stop the madness: uninstall PHP from your servers. We can do better than that!

## related articles

- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [meta data](/archives/2010/12/13/meta-data)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)
- [service update](/archives/2023/12/14/service-update)
- [colophonwards](/archives/2023/12/05/colophonwards)

### 3 responses

1. Henry says:[23 April 2008 1:29 PM](#6ef22a5977d377235135d76e4b79222180d0321c)

   So... your blogging software uses git as its backing store, but you can only check it out from bzr?

   Madness! :P

2. wingo says:[23 April 2008 2:07 PM](#fa46766cbace0a97007ed85f441336fb6a343e13)

   Madness indeed!

3. [Dread Knight](http://www.freezingmoon.org) says:[23 April 2008 3:54 PM](#7ecfc6a1599dbb4d0a5f0e5020aea1062dc11bad)

   MADNESS?! THIS IS SPARTA! xD

Comments are closed.
