---
title: Throw Up a Quick HTTP Server
url: https://nullprogram.com/blog/2010/09/21/
published: "2010-09-21T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/09/21/
---

# Throw Up a Quick HTTP Server

## [Throw Up a Quick HTTP Server](/blog/2010/09/21/)

September 21, 2010

nullprogram.com/blog/2010/09/21/

Ever since I learned [this neat trick from Luke](http://www.terminally-incoherent.com/blog/2010/04/08/exchanging-files-over-the-network-the-easy-way/) a few months ago I've been using it almost weekly. If you have Python installed then you have a miniature web server at your fingertips.

```
python -m SimpleHTTPServer

```

Bam! If I need to transfer large files directly to someone over the Internet, I'll throw up one of these and give them the address. Each of my home computers has an 8 *xxx* port forwarded to it in my router's NAT configuration, so they're all ready to do this any time I need it.

When I needed this before, I would use my own [Emacs web server](/blog/2009/05/17/), but the Python solution above is even more convenient most of the time. Plus it does directory listings, which I never bothered to add to my web server.

The reason I bring it up right now is because it finally saved me a lot of time at work. I was performing a half dozen Ubuntu installs for a system we're building, but none of these computers have network access, except to each other. I was using the Ubuntu DVD, which includes a larger software selection than the CD.

Well, it seems no one ever actually uses the DVD like this, because it doesn't work right now with the current DVD. You see, `apt` doesn't treat the DVD as just any repository. It has its own special cache for them, keeping track of what packages are on what CDs and DVDs. Well, the problem is that the CD is named differently than the DVD, and `apt` (and `apt-cdrom`) keeps looking for the CD when it should be looking for the DVD, foolishly ignoring any advice I give in the configuration. Python SimpleHTTPServer to the rescue!

I mounted the DVD and ran the HTTP server at the DVD's root. Then I added my localhost server to the `sources.list`. Worked like a charm! `apt` had no idea it was pulling packages off the DVD.

- [trick](/tags/trick/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Throw%20Up%20a%20Quick%20HTTP%20Server) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Throw+Up+a+Quick+HTTP+Server).

« [Middleman Parallelization](/blog/2010/09/08/)

» [Elisp Higher-order Conversion to Interactive](/blog/2010/09/29/)
