---
title: cypherpunks writecode — wingolog
url: https://wingolog.org/archives/2006/07/21/cypherpunks-writecode
published: "2006-07-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/07/21/cypherpunks-writecode
---

# cypherpunks writecode — wingolog

## [cypherpunks writecode](/archives/2006/07/21/cypherpunks-writecode)

22 July 2006 1:42 AM

- [advogato](/tags/advogato)
- [gstreamer](/tags/gstreamer)
- [streaming](/tags/streaming)
- [ogg](/tags/ogg)

Because I egocentrically syndicate myself on [advogato](http://advogato.org/recentlog.html?thresh=3), which only shows your most recent post, I've shied away from multiple postings at a time. The result is mashing a bunch of topics into one writing product. This practice has its, um, aspects.

The biggest minus for me is that I have to have all of my thoughts finished at the same time. I'm going to experiment with shorter writings on marginally more focused topics. We'll see how it goes; advo folks might want to look at my [last entry](http://advogato.org/person/wingo/diary.html?start=166), which is more advo-related.

**hack hack hack!**

Today was Friday at [Fluendo](http://fluendo.com/). Although I do bitch a lot about various things work-related, the fact that we have a day a week to hack on what we like is hot. It's cunning though, being at the end of the week, as the momentum of the week's projects bleed a bit onto the last day, but still. Hack hack.

So today my goal was to figure out what was up with the [GUADEC](http://guadec.org/) video and audio archives. People are all up on my case about this, and wha? I just don't have the time I used to. (Reasons for this are in a book I'm working on.)

Anyway, so this is a prelude to the sequel to [my last post on conference streaming](//wingolog.org/archives/2006/07/07/so-you-want-to-stream-a-conference). What I wanted to say was something about ensuring that you get archives on disk, and then to edit them later.

However I was running into problems. Sometimes totem was having trouble playing the files, for example. Wha? Also there were some sound level issues. Amusingly, in the sala d'actes, an unbalanced cable converter was picking up the radio, for example.

More disturbingly, all players (gstreamer, xine, mplayer, vlc) were playing unsynchronized audio and video for some talks. *How could this be? I was getting proper timestamps from the DV feed, but somehow in the encoding process we are producing bad ogg timestamps (granulepos values)? Wha?* The italics totally indicate internal dialog.

A little background: the [2005 Guadec video archives](http://stream.fluendo.com/archive/6uadec/) suck. I say this having been a part of the process of their creation. Most do not pass [oggz](http://www.annodex.net/software/liboggz/html/)-validate. This is because of problems in the [GStreamer](http://gstreamer.freedesktop.org/) ogg muxer back then.

Oh man we were pissed, in the American sense. How could we produce bad ogg. Us the exponents of ogg. Gar. So [Thomas](http://thomas.apestaart.org) fixed the ogg muxer For Once And For All, and the world was happy.

Another background: the way that we produced those videos was by watching the talks, then when a new speaker started, we pressed a button in a [flumotion](http://flumotion.net)-admin client that we had running, which would tell the "disker" flumotion component to start a new file. Because we didn't have any decent cutting tools, we had to rely on this to produce files per-talk. It was a bit of work, but it produced decent results. At least we could do it from anywhere with network access.

Fast forward to 2006. I had streamed a couple of conferences, and thought that it was a pain in the ass to have to have someone rotate the conference archives manually. This was a selfish desire, that although I was the person responsible for streaming, I wanted to enjoy the conference too -- always having to rotate the videos is a drag. So I wrote a [lossless cutter for ogg/theora+vorbis](//wingolog.org/archives/2006/05/26/tape-ate-the-player), the intention being to let the video capture to disk all the time, then just cut out the talks you want.

So I cut the talks. I get some segfaults, patch some code, update to latest CVS, have to patch it some more, but in the end I get cuts which I believe to be correct. Only problem is, the audio is completely off. As in, 10 seconds out of sync. *This should not be possible*. I mean, my GStreamer talk (not yet posted) was about synchronization. I know how to do this. What was up?

Well. Long story short, after despairing to [Thomas](http://thomas.apestaart.org/), we figure that if the CPU usage spikes, such that the theora encoder takes too long and we get behind, that it could be that we have to drop frames. This will happen on the capture end of things, if what is processing the raw data is not reading fast enough. In theory this is fine. The encoder will still receive correctly-timestamped data. However, GStreamer's Vorbis and Theora encoders were internally assuming perfect streams (no dropped data), so internally they disregarded the timestamps they were given, choosing instead to produce continuous streams.

The end result is that our GUADEC archives are perfect, in one sense: they present no problems for oggz-validate or for ogginfo (the two programs you should use to validate ogg files). However they are incorrect. In the event of dropped data, the audio and video become unsynchronized.

This is even more of a problem for the long files I chose to record. What a PITA. I have broken files that I will need to manually patch at certain points to resync the vorbis and theora ideas of granulepos. This means we need even more custom tools. Ug.

So the end is that seekers of GUADEC archives will have to wait a bit.

**what I meant to say**

Writecode. Indeed what I meant to say was that after recognizing the bogus behavior of GStreamer's vorbis and theora encoders, that Mike Smith and I set out to fix them. He hacked up patches while I hacked unit tests. All I wanted to say was that it was really nice to hack GStreamer, after so long away, and that C is fun sometimes.

**also wha is the new what**

## related articles

- [whoisi impressions](/archives/2008/11/14/whoisi-impressions)
- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [sunset skies; a plea for help; an outgrowing](/archives/2008/06/25/sunset-skies-a-plea-for-help-an-outgrowing)
- [catching memory leaks with valgrind's massif](/archives/2008/05/05/catching-memory-leaks-with-valgrinds-massif)
- [(pop! stack) => 'guadec-archives](/archives/2007/01/19/pop-stack-guadec-archives)
- [why god made buttermilk](/archives/2006/12/17/why-god-made-buttermilk)

### 5 responses

1. says:[22 July 2006 3:33 AM](#150)

   Would it be possible to initially just publish the audio?

2. [wingo](http://wingolog.org/) says:[22 July 2006 10:54 AM](#151)

   Anonymous: Sure! That would be much easier. I'll see about doing that.

3. [Ciaran O'Riordan](http://ciaran.compsoc.com) says:[4 August 2006 6:54 PM](#169)

   I came here to suggest publishing the audio first, but someone's beaten me to it :-)

   I was an organiser of FSFE's GPlv3 conference a few days before GUADEC. I'm no expert, but in case it's any use, the guy who did our DV to Theora+Vorbis conversion (Sean Daly), used this command:

   $ date ; ./ffmpeg2theora /Volumes/Medias/Barcelona/GPL3Barcelona\_rms.dv -x 352 -y 288 -v 2 -\\S 0 -K 128 -c 1 -H 32000 --artist 'Richard M. Stallman' --title 'Overview of GPL v3 Changes' -\\-date 'June 22, 2006' --location 'Third International GPLv3 Conference, Barcelona' --organization \ 'Free Software Foundation Europe (http://www.fsfeurope.org)' --copyright '(c) 2006 Free Software F\\oundation Europe' --license 'Verbatim copying and distribution of this video or any portio\\n thereof is permitted worldwide, without royalty, in any medium, provided this notice, and the copy\\right notice, are preserved.' -o /Volumes/Medias/Barcelona/GPL3Barcelona\_rms.theora.ogg ; date

   ...and he found useful info here: http://www.v2v.cc/~j/ffmpeg2theora/

   (I know you're an experienced Ogger - but I figured there's no harm in passing this on.)

   Here are the videos:

   http://fsfeurope.org/projects/gplv3/barcelona-summaries

4. [wingo](http://wingolog.org/) says:[4 August 2006 7:09 PM](#170)

   Hey Ciaran, thanks for the pointers :) I don't think I did a good enough job to push my way as the only way to do things.

   Actually I was quite impressed with what [these people](http://free-electrons.com/communaute/videos/community/videos/conferences) are doing; not being live they avoid some of the combinatoric problems I had at guadec.

5. [the world is our clique -- andy wingo](http://wingolog.org/archives/2006/08/05/the-world-is-our-clique) says:[5 August 2006 10:17 PM](#171)

   \[...\] Oh, the ongoing saga of Friday. I suppose if you’re reading this, hoping to pick up some tips about video recording and streaming, my first advice would be to forget about streaming. I look at what the free electrons folks are doing, and am quiite jealous of their output. \[...\]

Comments are closed.
