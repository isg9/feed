---
title: the user in the loop — wingolog
url: https://wingolog.org/archives/2011/10/19/the-user-in-the-loop
published: "2011-10-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/10/19/the-user-in-the-loop
---

# the user in the loop — wingolog

## [the user in the loop](/archives/2011/10/19/the-user-in-the-loop)

19 October 2011 2:32 PM

- [gnu](/tags/gnu)
- [ghm](/tags/ghm)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [extensibility](/tags/extensibility)
- [alexander](/tags/alexander)
- [c](/tags/c)

The videos from this year's [GNU Hackers Meeting](http://www.gnu.org/ghm/2011/paris/) in Paris are up. All the videos are linked from that page. There were technical problems with a couple of them, and we didn't get BT Templeton's presentation on Emacs Lisp in Guile (using delimited dynamic bindings to implement buffer-local variables!), but all in all it's a good crop.

I gave a talk entitled "The User in the Loop" that argued for the place of extensibility in the GNU project. I also pointed out some ways in which Guile fulfills those needs, but that was not the central point to the talk. I argued that point at more length [in a previous article](//wingolog.org/archives/2011/08/30/the-gnu-extension-language).

Anyway, let's give that newfangled HTML5 video tag a go here. It starts with the cut-off statement, "We all know that code motion is very important to efficiency, so..."

[Click to watch](http://audio-video.gnu.org/video/ghm2011/Andy_Wingo-_The_user_in_the_loop.ogv)

Alternately you can [download the video](http://audio-video.gnu.org/video/ghm2011/Andy_Wingo-_The_user_in_the_loop.ogv) directly (~350MB, 52 minutes). There are [notes](http://www.gnu.org/ghm/2011/paris/slides/andy-wingo-guile-notes.pdf) too, a superset of the [slides](http://www.gnu.org/ghm/2011/paris/slides/andy-wingo-guile.pdf) from the talk.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 4 responses

1. Brandon says:[20 October 2011 5:31 AM](#c76e85cc4e5d815b4c3959f1b6beb35f127b75fe)

   I just wanted to say that I enjoyed watching your talk, so thanks for posting it. Lately I've been nostalgic for the time I spent in Barcelona, and your last couple of posts are surely not helping ;)

2. Brandon says:[20 October 2011 5:52 AM](#1a83100f01affd72785e80540e8284b511728398)

   Also wanted to comment on the the nature of renting, which comes up during your talk. I think I understand what Gabriel means. I value the freedom that comes with renting, but that freedom comes at a price. That price being that, though I'd love to change them, there are things that drive me crazy: The cheap-looking stain on the cabinetry, the hideous formica on the bar, the decades-out-of-fashion hardware that doesn't match anything I own. These things could be fixed in a weekend, but verboten for me to change them, quite apart from the fact that I am reluctant to spend my own resources improving another man's property.

   I have a book called "nomadic furniture", full of DIY design projects that are work-arounds for things like this. There are plans for stacking book-cases, hanging shelves, closet inserts that brace against the ceiling with screw-fittings, collapsible tables and chairs, and other various solutions to the problem of changing a space that you will occupy but temporarily. The book is a lovely cultural relic from the 1970s, but, sadly, predicated on some faulty assumptions: that plywood is an inexpensive building material; and that landlords won't mind many dozens of holes in the walls, floors, ceilings etc. Moreover, building furniture from scratch within the confines of my studio apartment would be very much like programming an extension in an embedded environment on an 8.9-inch eeePC ;)

3. [Ramakrishnan Muthukrishnan](http://rkrishnan.org) says:[20 October 2011 11:46 AM](#ab5fc69231cf75e5248ebec101c27b3d9770d81e)

   Really enjoyed the talk. I definitely second your opinions on copyright assignment. I have no problem assigning copyrights to the FSF and have done that before. But it is \*hard\* to get the documentation done from employers and I have been a victim of bureaucracy.

4. [Alaric Snell-Pym](http://www.snell-pym.org.uk/alaric/) says:[20 October 2011 1:07 PM](#835b5769f0d11098c90b48d3d5f0f3dce7b8ec11)

   I'm a big fan of building applications around a "plugin" model; I've found myself that if you take a monolithic app, and add plugins for some isolated extensibility point, then within a few months you find yourself realising that all sorts of bits of the system can be chopped off and turned into plugins, simplifying your software architecture and tremendously increasing the user-extensibility of your app...

   Here's a blog post I wrote on the matter on 2006, a fact which has just made me feel tremendously old: http://www.snell-pym.org.uk/archives/2006/09/15/plugin-based-applications/

Comments are closed.
