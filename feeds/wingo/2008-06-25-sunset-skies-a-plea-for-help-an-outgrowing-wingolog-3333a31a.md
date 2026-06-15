---
title: sunset skies; a plea for help; an outgrowing — wingolog
url: https://wingolog.org/archives/2008/06/25/sunset-skies-a-plea-for-help-an-outgrowing
published: "2008-06-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/06/25/sunset-skies-a-plea-for-help-an-outgrowing
---

# sunset skies; a plea for help; an outgrowing — wingolog

## [sunset skies; a plea for help; an outgrowing](/archives/2008/06/25/sunset-skies-a-plea-for-help-an-outgrowing)

25 June 2008 9:24 PM

- [gnome](/tags/gnome)
- [the sky](/tags/the%20sky)
- [intel](/tags/intel)
- [advogato](/tags/advogato)
- [conversations](/tags/conversations)

Good evening!

Here the evening is good indeed -- tepid air, the kind where you don't know it's there until it moves, or you move: skin temperature. We don't get too many colored cloud sunsets, but today there was sky-drama over the wasteland of [la Sagrera](http://es.wikipedia.org/wiki/Barrio_de_La_Sagrera).

**q**

I have a question, to those in the know. It's nargery. I have a machine with intel graphics, a GM965. It claims to support the texture\_from\_pixmap extension, but I have not been able to make it work, neither in my own code nor in that of others. I whine about it more [in the fedora bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=452915). What's the dilly?

**conversations**

I'm in a fortunate position to have readers that might just be able to answer that question. This is mostly a result of being syndicated on [Planet GNOME](http://planet.gnome.org/), although Advogato and individual subscribers do play a role. But it is good to write for an audience.

In that regard, I think that [Advogato](http://advogato.org/recentlog.html) is a much more appropriate system for publishing and reading, for conversation, than are the various [planets](http://planetplanet.org/) out there. Advo has no barrier to entry; [anyone can start writing](//wingolog.org/archives/2002/05/31/advogato-0). You can syndicate your writings from elsewhere, like you can with a planet, or write them there if you feel like it. If your writings are good, or interesting, then folks will read them. If some people don't like them, they don't have to see them. If no one likes them, no one will see them.

Granted, present-day advogato does have its problems. It seems that its model of trust has drifted too far from the root nodes; my ratings never affect someone's rank, even to go from observer to apprentice. "Apprentice", while it does correspond to reality in some sense, is seen as demeaning; everyone wants to be a journeyer, if not a master. The front-page articles have been of low quality for a long time. But the diary "interesting-ness" ratings do seem to have stood the test of time.

Project-specific aggregators like Planet GNOME are great. They're wonderful for community, and good for communication too: "this is who we are". But I do miss the easy anarchy of Advogato. Maybe it's time to make an antiplanet, running on mod\_virgule. Thoughts?

## related articles

- [whoisi impressions](/archives/2008/11/14/whoisi-impressions)
- [rococo, and then rubble](/archives/2012/06/08/rococo-and-then-rubble)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [to guadec!](/archives/2011/08/03/to-guadec)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [gnu, gnome, and the fsf](/archives/2009/12/13/gnu-gnome-and-the-fsf)

### 3 responses

1. Paul Collins says:[25 June 2008 11:18 PM](#f26a3aa0a806a5ec5bf450c6dc9b10e7f0c0baa3)

   The desktop effects thing could be red herring. For example, Metacity's compositor uses RENDER; no OpenGL at all. And COMPOSITE itself is a very simple set of operations on fundamental X server objects that doesn't say much (or maybe even anything) about how to assemble what ends up on the screen: http://cgit.freedesktop.org/xorg/proto/compositeproto/tree/compositeproto.txt

2. loicm says:[26 June 2008 0:03 AM](#2707eb801ea22d4c094ca98d08df523f594a6bb5)

   For your texture\_from\_pixmap issue, I think it could be due to two things.

   The first one being the support for the extension itself, with DRI drivers as it's your case you need to explicitely request indirect rendering setting LIBGL\_ALWAYS\_INDIRECT in the environment.

   The other tricky issue could be due to the use of non power-of-two redirected X11 drawables. Because if the GPU/driver doesn't support non power-of-two textures, it results to nothing rendered or as it's the case for the guy on the bugzilla with (white) non-textured polygons. To sum up, if you've got the GL\_ARB\_texture\_non\_power\_of\_two extension it means you can use GL\_TEXTURE\_2D as texture target with whatever texture size and texture coordinates in the range \[0.0, 1.0\]. If you don't have it, but got the GL\_ARB\_texture\_rectangle extension it means you can use GL\_TEXTURE\_RECTANGLE\_ARB as texture target with whatever texture size and texture coords in the range \[0.0, size\]. If you don't have one of these extensions, you'll only be able to use the GL\_TEXTURE\_2D texture target with power-of-two textures and texture coords in the range \[0.0, 1.0\].

   Hope that helps.

3. [wingo](http://wingolog.org/) says:[26 June 2008 9:53 AM](#fbaa97a36a7ff06e979de463e504f22144018bb0)

   Thanks Paul and Loic. LIBGL\_ALWAYS\_INDIRECT=1 does appear to get a first copy of the pixmap onto the screen, but was not updating when I tried this morning; will poke later on.

Comments are closed.
