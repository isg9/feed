---
title: dangling pointers — wingolog
url: https://wingolog.org/archives/2008/11/19/dangling-pointers
published: "2008-11-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/11/19/dangling-pointers
---

# dangling pointers — wingolog

## [dangling pointers](/archives/2008/11/19/dangling-pointers)

19 November 2008 10:38 AM

- [moon](/tags/moon)
- [lisp](/tags/lisp)
- [azul](/tags/azul)
- [oblong](/tags/oblong)
- [party barge](/tags/party%20barge)
- [silver jews](/tags/silver%20jews)
- [music](/tags/music)

I have here three pointers into the ether.

**with apologies to rubén darío**

[A conversation between Dave Moon, Dan Weinreb, and Cliff Click on Azul, a massive multiprocessor Java machine](http://blogs.azulsystems.com/cliff/2008/11/a-brief-conversation-with-david-moon.html).

The Azul is a Java machine like the Symbolics that Moon and Weinreb worked on was a Lisp machine. Easily the most interesting read of the last week or two, via [Michael Weber](http://www.foldr.org/~michaelw/log/programming/lisp/click-moon-weinreb).

This one will be on the final exam.

**house in order**

The company I work for, [Oblong Industries](http://oblong.com/), just opened up their web site on Friday. Go check it out!

I like how they hook visitors with the videos, then pull a [Yegge](http://steve-yegge.blogspot.com/2008/01/blogging-theory-201-size-does-matter.html) with all of the text they have there. I approve!

**on repeat**

[Silver Jews](http://www.silverjews.net/)' latest album, *Lookout Mountain, Lookout Sea*, is quite addictive. It's no *American Water* but I can't get enough of [Party Barge](http://www.last.fm/music/Silver+Jews/_/Party+Barge).

*Send us your coordinates, I'll send a Saint Bernard...*

## related articles

- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)
- [new beginnings](/archives/2011/04/05/new-beginnings)
- [ports, weaks, gc, and dark matter](/archives/2011/02/25/ports-weaks-gc-and-dark-matter)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [shoptalk](/archives/2010/06/21/shoptalk)
- [rememberings](/archives/2009/10/22/rememberings)

### 3 responses

1. [Colin Walters](http://cgwalters.livejournal.com) says:[19 November 2008 2:26 PM](#8805ec7b4cf8b8888c8b4c9c784f698cf224c5fe)

   Yeah, the Azul boxes are pretty fascinating. One does wonder when tighter VM-OS-hardware integration will start to become mainstream. On general purpose mainstream OSes the first pair is generally pretty sad, Android being a notable exception.

   What amazes me the most about the Azul systems though is how good they made GC. 10 Gigs/second of garbage and it doesn't break a sweat.

2. [Andy Wingo](http://wingolog.org/) says:[19 November 2008 2:49 PM](#06a732a01333a8fcae18f94ce1f52eb5b793e420)

   Yeah Colin, I'm quite in awe of a number of things that they did. The wide tagging allows them to encode the generation into the pointer directly -- and indeed encode *the class itself* into the pointer, which is pretty crazy.

   Now, if only the JVM had tail calls...

3. Pete says:[18 December 2008 9:40 AM](#2435aec1114437aec29d3db09df45f7ce537659c)

   The oblong video is pretty cool even though I get the impression you can't really do anything productive with it yet. Shooting floating Chinese characters did look like a lot of fun.

Comments are closed.
