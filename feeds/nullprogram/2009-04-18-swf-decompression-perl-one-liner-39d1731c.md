---
title: SWF Decompression Perl One-liner
url: https://nullprogram.com/blog/2009/04/18/
published: "2009-04-18T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/04/18/
---

# SWF Decompression Perl One-liner

## [SWF Decompression Perl One-liner](/blog/2009/04/18/)

April 18, 2009

nullprogram.com/blog/2009/04/18/

![](/img/misc/magnify.png)

Flash seems to be the popular way of playing videos online. This is a bit better than the bad old days of online video where a user had to select from a few buggy media player plug-ins. Things have improved.

However, if you don't use Flash, or if you want to watch the videos in your own media player, you are stuck. A download link for the video is almost never provided. The video is always somewhere, though, to be fetched via http. I [mentioned this
before](/blog/2007/09/05) for downloading YouTube videos using [youtube-dl](http://rg3.github.com/youtube-dl/).

The trick is finding the URL. Sometimes you can derive it from the HTML code, sometimes you have to dig a little deeper by inspecting the Flash player itself. [`\ strings`](http://en.wikipedia.org/wiki/Strings_(Unix)) can be invaluable here.

There could be an extra layer of stuff to work out, which is explained below. My main reason for posting this is so I can refer back to it later when I need to do it again.

So, the other day I ran into a Flash video player that contained part of the URL of its video. I began by studying the `embed` tag in the HTML, which gave me some information about where to find the video (the video ID number). I downloaded the Flash player SWF file for the purpose of running `strings` on it.

I ran into a problem here. I wasn't finding any non-garbage strings inside the file. `file` told me it was *compressed*.

```
$ file player.swf
player.swf: Macromedia Flash data (compressed), version 9

```

Searching online quickly revealed that a compressed Flash file is just zlib compression after an 8-byte header. Decompression can actually be done with a Perl one-liner,

```
perl -MCompress::Zlib -0777 -e \
      'print uncompress substr <>, 8;' player.swf > player

```

I ran `strings` and `grep` ed for "http", revealing the location of the video. That was it!

I actually came across [a Java program](http://www.brooksandrus.com/blog/2006/08/10/) that does the same thing. It is 115 lines of code. Java programs always seem to be bloated like this.

I hope you find this useful!

- [perl](/tags/perl/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20SWF%20Decompression%20Perl%20One-liner) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=SWF+Decompression+Perl+One-liner).

« [URL Shortening](/blog/2009/04/16/)

» [A Not So Stupid C Mistake](/blog/2009/04/20/)
