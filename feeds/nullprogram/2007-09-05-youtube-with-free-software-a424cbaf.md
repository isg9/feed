---
title: YouTube with Free Software
url: https://nullprogram.com/blog/2007/09/05/
published: "2007-09-05T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2007/09/05/
---

# YouTube with Free Software

## [YouTube with Free Software](/blog/2007/09/05/)

September 05, 2007

nullprogram.com/blog/2007/09/05/

**Update 2009-6-30**: *Thanks to HTML 5 and the `video` tag I will be* *self-hosting videos from now on. This is information is only* *historical.*

As I have stated previously, I love [free software](http://www.fsf.org/) and I try to use free software exclusively whenever I can (it is very difficult to find employment in computer engineering where no proprietary software is used). This can pose a problem when I want to watch [YouTube](http://www.youtube.com/) videos because I do not use the proprietary, non-free Flash player. The free Flash players currently either handle YouTube poorly or not at all. I also find Flash annoying enough that I am not interested in using these free Flash players anyway (fewer ads automatically!).

Like everyone else with an e-mail address, I get links to videos on YouTube from my friends. I also post videos there myself under the name “throwaway0” as it is convenient not only for me, but also for anyone who wants to watch the videos. Now, if the only way to watch these videos was with proprietary software, I would not encourage this. In fact, not too long ago this was true of most online video, which was limited to a “choose your poison” type situation between the proprietary, worthless Windows Media and QuickTime formats. No poison for me, thanks.

I have discovered several solutions to watching YouTube with free software. There are two steps involved: getting the video and playing the video. For the first you have [youtube-dl](http://rg3.github.io/youtube-dl/)and then you have [Firefox](http://www.mozilla.com/en-US/firefox/) [Fast Video Download](https://addons.mozilla.org/en-US/firefox/addon/3590).

youtube-dl is a Python script that you can easily install on your system. You just give it a YouTube URL and it does all the work. It feels a bit like wget. The Firefox add-on will add a nice little icon like this,

![](/img/fast-video-download.png)

Clicking this icon will download the video. With either tool, you will have this .flv file somewhere that you want to watch.

To watch the video file, you can use [mplayer](http://www.mplayerhq.hu/) or [VLC](http://www.videolan.org/vlc/). As far as I know, these videos are handled with free software only when using these players. They play fine (except without seeking) on my system. I use [Debian GNU/Linux](http://www.debian.org/) so I am pretty confident that my system is strictly free software.

Now you can watch YouTube without having to fall victim to proprietary software.

- [rant](/tags/rant/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20YouTube%20with%20Free%20Software) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=YouTube+with+Free+Software).

« [Mandelbrot with GNU Octave](/blog/2007/09/02/)

» [PNG Archiver - Share Files Within Images](/blog/2007/09/08/)
