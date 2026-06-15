---
title: the world is our clique — wingolog
url: https://wingolog.org/archives/2006/08/05/the-world-is-our-clique
published: "2006-08-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/08/05/the-world-is-our-clique
---

# the world is our clique — wingolog

## [the world is our clique](/archives/2006/08/05/the-world-is-our-clique)

5 August 2006 10:17 PM

- [guadec](/tags/guadec)
- [streaming](/tags/streaming)
- [iran](/tags/iran)

**anger**

[![](http://www.zmag.org/iranwatch/lolita.jpg)](http://www.zmag.org/content/showarticle.cfm?SectionID=67&ItemID=10707)

> And then who gets to review it for the Wall Street Journal, certainly not Mahmoud Omidsalar, but yet another neocon artist Azar Nafisi wannabe named Roya Hakakian who believes Dick Davis’ translation is not good enough. Dick Davis sneezes and there is more poetry and scholarship in it than in Roya Hakakian and seven generation of her parentage put together—and who died and made Roya Hakakian an authority either on Shahnameh or on its English translation?

Hamid Dabashi expounds adjectively in [Lolita and beyond](http://www.zmag.org/content/showarticle.cfm?SectionID=67&ItemID=10707) on ZNet. Enjoyable read if you're into that kind of thing.

**friday**

Oh, the ongoing saga of [Friday](//wingolog.org/archives/2006/07/21/cypherpunks-writecode). I suppose if you're reading this, hoping to pick up some tips about video recording and streaming, my first advice would be to forget about streaming. I look at what the [free electrons folks](http://free-electrons.com/community/videos/mini-howto) are doing, and am quiite jealous of their output.

The reason their output is better than what I have for guadec is threefold:

- They capture to raw DV video, which is very easy to edit. The tradeoff is that it takes a lot of space, but that isn't so much of a problem these days.

- They don't stream the video live, which decreases the combinatoric coordination clusterfuck that is [live conference streaming](//wingolog.org/archives/2006/07/07/so-you-want-to-stream-a-conference).

- They don't encode or process in realtime, which avoids the heavy and variable cost of theora encoding. Also the way that overlays are implemented in Flumotion right now is expensive.

So if you're looking to record video for something, record to DV, potentially even on a tape, and process it later. Very easy. Few things that can go wrong (sound, basically).

**but guadec archives?**

As a person who prides himself on the quality of his work, editing the GUADEC videos is depressing to the point of inaction-inducing. After a bit of diversionary LADSPA hacking this Friday I synced up with the videos that [Jamie](http://jaime.hemmett.org/blog/) had been cutting, about four or five talks. Jeff Waugh gives his opening guadec presentation, walks off the stage and cleans up, and two minutes later he's still talking on the audio track as [Maciej Katafiasz](http://mathrick.org/) wanders in front of the camera in an empty room. Terrible! Doubly so, because I had a hand in the software that made those archives.

At first I thought I could just get away with synchronizing the beginnings of the talks, but some videos show worse disconts than that. So, in the end I ended up modifying my [lossless ogg theora cutter](//wingolog.org/archives/2006/05/26/tape-ate-the-player) to produce a stream resynchronizer:

[![](//wingolog.org/wp-content/ogg-theora-resynchronizer.png)](http://cvs.freedesktop.org/gstreamer/gst-python/examples/synchronizer.py)

Now it relies on an unreleased version of gst-plugins-base, but in time things will be fine. Anyway, now that I have a decent process, hopefully I can rope my coworkers into cutting with me one day, and we just knock out all of the videos. Then I can write once more about all of this to say it's finished and proceed to some other, any other phase of my life.

**other**

In the oft-listen department, may I mention Deee-lite and Oumou Sangare. Thank you.

## related articles

- [(pop! stack) => 'guadec-archives](/archives/2007/01/19/pop-stack-guadec-archives)
- [passing through unconscious states](/archives/2006/07/10/passing-through-unconscious-states)
- [to guadec!](/archives/2011/08/03/to-guadec)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [guadec ho!](/archives/2009/07/02/guadec-ho)
- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)

### 5 responses

1. Greg says:[6 August 2006 1:29 AM](#172)

   Can you please put up the audio tracks (Without video). I would love to hear the talks and I can live without all that pacing and handwaving.

2. [Ricky Youngblood](http://www.macgregor26.com/) says:[6 August 2006 10:16 AM](#173)

   Can you please make it so that I can listen to the audio tracks on my [Mac 26](http://www.macgregor26.com/) while I'm drinkin' a cold one?

3. Munchkinguy says:[9 August 2006 3:02 AM](#177)

   Any guesses as to when you will have uploaded the GUADEC videos?

4. [wingo](http://wingolog.org/) says:[9 August 2006 12:48 PM](#181)

   Munchkinguy: Sunday perhaps? At least some of them by then.

   [World enough, and time](http://www.bartleby.com/101/357.html)...

5. Aladdin says:[14 August 2006 6:16 PM](#184)

   Time to release ? :)

Comments are closed.
