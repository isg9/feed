---
title: Ad-blocking and the Regrettable URL Format
url: https://nullprogram.com/blog/2009/08/02/
published: "2009-08-02T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/08/02/
---

# Ad-blocking and the Regrettable URL Format

## [Ad-blocking and the Regrettable URL Format](/blog/2009/08/02/)

August 02, 2009

nullprogram.com/blog/2009/08/02/

I use [Adblock Plus](http://adblockplus.org/en/) to block advertisements and, more importantly, invisible privacy-breaking trackers (most people aren't even aware of these). I think ad-blocking is actually easier than ever, because ads are served from a relatively small number of domains, rather than from the websites themselves. Instead of patterns matching parts of a path, I can just block domains.

Adblock Plus emphasizes this by providing, by default, a pattern matching the server root. Example,

```
http://ads.example.com/*

```

But sometimes advertising websites are trickier, and their sub-domain is a fairly unique string,

```
http://ldp38fm.example.com/*

```

That pattern isn't very useful. I want something more like,

```
http://*.example.com/*

```

Unfortunately Adblock Plus doesn't provide this pattern automatically yet, so I have to do it manually. I think this pattern is less obvious because the URL format is actually broken. Notice have have two matching globs (\*) rather than just one, even though I am simply blocking everything under a certain level.

Tim Berners-Lee [regrets the format of the URL](http://en.wikipedia.org/wiki/URL#History), and I agree with him. This is what URLs like `http://ads.example.com/spy-tracker.js` *should* look like,

```
http://com/example/ads/spy-tracker.js

```

It's a single coherent hierarchy with each level in order. This makes so much more sense! If I wanted to block example.com and all it's sub-domains, the pattern is much simpler and less error prone,

```
http://com/example/*

```

To anyone who ever reinvents the web: please get it right next time.

**Update**: There is significant further discussion in the comments.

- [rant](/tags/rant/)
- [web](/tags/web/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Ad-blocking%20and%20the%20Regrettable%20URL%20Format) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Ad-blocking+and+the+Regrettable+URL+Format).

« [Television Commercials](/blog/2009/07/28/)

» [Web Pages Are Liquids](/blog/2009/08/05/)
