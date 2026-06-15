---
title: stackpusher — wingolog
url: https://wingolog.org/archives/2002/06/08/advogato-1
published: "2002-06-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2002/06/08/advogato-1
---

# stackpusher — wingolog

## [stackpusher](/archives/2002/06/08/advogato-1)

8 June 2002 7:28 PM

- [music](/tags/music)

**Jack**

I've been tracking down a bug in [Jack](http://jackit.sf.net/) recently; it prevents [GStreamer](/proj/GStreamer/)'s Jack element from connecting Jack ports. It's a tricky multithreaded, multiprocess bug that's triggered by some thread interlocking and timing issues. It's taking a bit of time.

**beatbox**

I've been working on [beatbox](http://ambient.2y.net/beatbox/) for over a year now. It's why I got started with GStreamer and with Jack, and things are finally looking good. I've got a new vision for it; once the Jack element is complete for GStreamer, beatbox will be a Jack-only mixer and effects processor. It will produce no audio, only mix together Jack streams, applying effects along the way. Should be pretty neat. Probably more on this in the future.

**the task stack**

It's funny how you (read: me :) want to do something, but you end up having to do some many other things first to enable you to do that thing. I want a scriptable, real-time music generation framework that can respond to ambient

sounds. So, one tool is a scriptable mixer for multiple audio streams. So, I need a framework to build that app. So

I start work with GStreamer, work on Jack a little, etc etc... pushing items onto the task stack. Once you finish a

goal, pop off the stack and return to what you were working on, until you realize you need to do something else first... so much ongoing, small-stepped effort that oftentimes never gets completed.

but damn, I want to make some thumping, swirling beats !

## related articles

- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [rememberings](/archives/2009/10/22/rememberings)
- [dangling pointers](/archives/2008/11/19/dangling-pointers)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)
- [random things](/archives/2007/11/08/random-things)
- [b-b-b-barcelona](/archives/2007/10/31/b-b-b-barcelona)

Comments are closed.
