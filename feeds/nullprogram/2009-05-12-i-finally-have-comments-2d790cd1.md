---
title: I Finally Have Comments
url: https://nullprogram.com/blog/2009/05/12/
published: "2009-05-12T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/05/12/
---

# I Finally Have Comments

## [I Finally Have Comments](/blog/2009/05/12/)

May 12, 2009

nullprogram.com/blog/2009/05/12/

Update: This post is referring to my old web hosting situation. I'm now using external comment hosting because my blog is now statically hosted.

I finally have a comment system, thanks to [pollxn](http://www.nukekiller.net/pollxn/), a [blosxom](http://www.blosxom.com/) comment system that actually works. There is a link to it, indicating the number of comments, in the bottom of each post. Try it out and say hello.

Unfortunately, pollxn doesn't have any sort of anti-spam or [CAPTCHA](http://en.wikipedia.org/wiki/Captcha) system. If you look around the Interwebs where other people are using pollxn, you will see everyone has their own little CAPTCHA thing. Well, I am not different. I hacked together my own to keep away automated spammers.

It selects words from the dictionary (of 40,000 words in this case) and encrypts them with Blowfish in CBC mode, with a unique IV each time. This is to passed to the user, who passes it to an image generator which decrypts the word and uses GD in Perl to render it, apply some transforms, and drop a line randomly over it. The user submits the guess of the image along with the encrypted version (hidden field), which is decrypted and compared on the other end. The same encrypted ID cannot be used twice, but thanks to the IV the same word *can* be used twice.

Here are some samples. If you hit refresh, they will render differently. ( *Update: not any more. These are just static examples* *now.*)

![CAPTCHA sample: ormolus](/img/captcha/a.png)![CAPTCHA sample: morons](/img/captcha/b.png)![CAPTCHA sample: softer](/img/captcha/c.png)![CAPTCHA sample: zanucks](/img/captcha/d.png)![CAPTCHA sample: grumble](/img/captcha/e.png)![CAPTCHA sample: nozzle](/img/captcha/f.png)

It's not a great CAPTCHA, but it should be good enough for the low volume of traffic I see here. As I inevitably collect small amounts of spam (by spammers manually passing the CAPTCHA), I will gradually create the needed tools to combat it. I can also easily update the CAPTCHA image algorithm without disrupting the functioning of the website.

I'm sure I will be making improvements to the comment system over time as well. I should make it obfuscate e-mail addresses, for one. Maybe add a preview. And better blosxom integration.

So say hello below! I am excited to finally have a *real* blog.

- [meta](/tags/meta/)
- [netsec](/tags/netsec/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20I%20Finally%20Have%20Comments) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=I+Finally+Have+Comments).

« [Greasemonkey User Scripts](/blog/2009/05/05/)

» [Emacs Web Server](/blog/2009/05/17/)
