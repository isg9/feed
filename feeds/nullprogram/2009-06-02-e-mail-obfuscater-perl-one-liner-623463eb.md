---
title: E-mail Obfuscater Perl One-liner
url: https://nullprogram.com/blog/2009/06/02/
published: "2009-06-02T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/06/02/
---

# E-mail Obfuscater Perl One-liner

## [E-mail Obfuscater Perl One-liner](/blog/2009/06/02/)

June 02, 2009

nullprogram.com/blog/2009/06/02/

If you look at the page sources around here you might notice that there are no bare e-mail addresses around. This is because I obfuscate them into a series of HTML entities. So far this has been pretty effective at hiding from address-scraping, web-crawling spam bots. They don't seem to try very hard at decoding HTML entities.

When I added the comment system, I needed to obfuscate addresses automatically. I quickly realized that this is yet another perl one-liner (and implemented as one line in the comment system). It can be used on the command line to obfuscate a file/pipe containing a list of addresses.

```bash
perl -lpe '$_ = join "", map {"&#" . ord() . ";"} split //'
```

All of the spaces are really only there for us humans,

```bash
perl -lpe '$_=join"",map{"&#".ord.";"}split//'
```

I keep running into these one-liners.

- [perl](/tags/perl/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20E-mail%20Obfuscater%20Perl%20One-liner) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=E-mail+Obfuscater+Perl+One-liner).

« [Elisp Wishlist](/blog/2009/05/29/)

» [JavaScript Distributed Computing](/blog/2009/06/09/)
