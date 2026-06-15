---
title: meta data — wingolog
url: https://wingolog.org/archives/2010/12/13/meta-data
published: "2010-12-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/12/13/meta-data
---

# meta data — wingolog

## [meta data](/archives/2010/12/13/meta-data)

13 December 2010 9:08 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [tekuti](/tags/tekuti)
- [meta](/tags/meta)
- [http](/tags/http)
- [web](/tags/web)

Hey tubes! Long time no type, in this direction at least.

It seems that most of my writing energy these past few months has been directed towards [Guile](http://www.gnu.org/software/guile/). For example, right now I should be writing documentation for new hacks, but instead I am typing at another part of the ether.

It's good and bad, this thing. The good thing is the hack-cadence in Guile is high. The bad thing is that not many learn about it, because, well, code doesn't blog about itself, does it?

[![](http://2.bp.blogspot.com/_D_Z-D2tzi14/TNiVf51_dfI/AAAAAAAAEDc/gf67HrwlAi8/s400/dogs26altalt.png)](http://hyperboleandahalf.blogspot.com/2010/11/dogs-dont-understand-basic-concepts.html)

Except in this case, perhaps. The tin can jiggling the electrons at the other end of this blogline has been my hack, of late. What you are reading is words about Scheme web servers, served by a Scheme web server.

[![](http://2.bp.blogspot.com/_D_Z-D2tzi14/TNiRrq56_fI/AAAAAAAAEDY/iaaOLq3S6N4/s400/dogs30alt.png)](http://hyperboleandahalf.blogspot.com/2010/11/dogs-dont-understand-basic-concepts.html)

That's right, I ported [Tekuti](//wingolog.org/software/tekuti/) to Guile 2.0. Delicious dogfood, yum!

In the process, I decided that [mod-lisp](http://www.fractalconcept.com/asp/53I1/sdataQGxMQy5X-vAqDM==/sdataQuvY9x3g$ecX), which I had been using, was stupid. There is already a simple, standard way of serving HTTP requests over a socket, and it is HTTP. So I wrote pieces of a web server, and put them in Guile. I'll probably write more about that later, so no more words about that for now, except to request that folks with spiders, bots, odd rss grabbers and such send me bug reports if things aren't legit.

**ciao slicehost, ciao linode**

About the same time, my bank decided to change my credit card, so all my old subscriptions stopped working. It was just the thing I needed to make me jump ship, finally, from [slicehost](http://slicehost.com/) to [linode](http://linode.com/).

If you're still on slicehost, I heartily recommend that you switch. (Heartily! Strange word. Like gravy and meatballs or something.) Linode feels faster to me, it's half the price, and otherwise the quality is about the same or perhaps a little better. And from what I hear, the linode offerings continue to improve, while slicehost hasn't changed for the 2+ years that I was with them.

Anyway, rap at yall soon, and keep your parentheses warm in this at-times cold Northern winter. Peace!

*Images courtesy of the excellent [Hyperbole and a Half](http://hyperboleandahalf.blogspot.com/).*

## related articles

- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [on the new posix](/archives/2010/12/17/on-the-new-posix)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [an in-depth look at the performance of guile's web server](/archives/2012/03/08/an-in-depth-look-at-the-performance-of-guiles-web-server)
- [for love and $](/archives/2012/02/21/for-love-and-)

### 3 responses

1. Daniel Díaz says:[14 December 2010 6:29 AM](#f904481c6d71f2ab3aa4c940cb408a08790b0c21)

   I can also recommend ProVPS (dot com). They charge the same for the RAM as Linode, but give you more bandwidth and disk space.

2. [Mark Van den Borre](http://blog.markvdb.be) says:[14 December 2010 2:39 PM](#591f7d289f4d7a3b005715532d4ea563feb9a7d3)

   Hi Andy,

   I am a linode client, and quite happy about their technical quality and fanatical support. Not so impressed by their "no wikileaks mirrors" policy though.

   I was quite impressed by OVH's answer to pressure by French foreign minister Eric Besson. I might consider them for new virtual private servers...

3. Brandon says:[16 December 2010 7:49 PM](#2bacff5b07b4791eebab7cc4410eb83276c4e620)

   A terrible fact has come to my attention: Google currently returns no matches for the phrase "Why guile is awesome" (with quotes). It does return matches for the phrase "Guile is awesome" (with quotes), however nothing within the first couple of pages that is not Street-Fighter-related (also awesome, but for different reasons). For posterity's sake, I suggest that you use your next blog posting to proclaim loudly that which we already know to be true.

Comments are closed.
