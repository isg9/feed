---
title: August fulcrum — wingolog
url: https://wingolog.org/archives/2005/08/17/august-fulcrum
published: "2005-08-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/08/17/august-fulcrum
---

# August fulcrum — wingolog

## [August fulcrum](/archives/2005/08/17/august-fulcrum)

17 August 2005 4:00 AM

- [random](/tags/random)

If I were a high school student in the north of Namibia, I would begin writing like this:

Dear diary, I write you in greetings. How is the situation on your side? Here I am calm and cool like a fish in the sea.

Formulaic but goofy. As I am not 16, in addition to being a native English speaker, I begin differently.

**acontecimientos**

I did indeed go to [Sanfermines](//wingolog.org/archives/2005/07/08/updates). It was a large party. The best thing about it for me was how wonderful the atmosphere was. At one point we were in a plaza where people were throwing themselves off a statue, a kind of stage-diving at higher heights and instead of a stage a phallic monument. Wild.

Standing there watching, the streets packed with people, we hear a whistling coming from farther up the hill. A few minutes later the whistles arrive. One was held by a guy in jeans, sandals, and a dirty football jersey. He carried a rickety-looking metal structure made to work as a miniature goal, maybe 2,5m by 3,5m. He placed it in front of a door in the plaza while his buddies, dressed referee-style in turf shoes, uniforms and lanyards, opened up a space for the penalty shootout.

The first fellow to get up there put it in, to the raucous delight of the crowd. After that there were a few misses, in between which the ref ran around trying to keep the area open.

At one point a fellow took the ball and kicked it on his own, without authorization. Clearly this was grounds for ejection. The ref blew his whistle and pointed down the street. A bystander came to his aid, holding up a flattened paper coca-cola cup as a provisional red card.

It ended about 45 minutes later, with ample numbers having their shot on goal. Drunk, nutty, all in good fun.

I just wanted to get to the part about the red card. Thanks for bearing with me.

**other trips**

I was down in Valencia for [Campus Party](http://web5.campus-party.org/index.php3), streaming the talks in the Software Libre area. I also gave a presentation on Flumotion, in Spanish. Will get the archives up shortly. There is a need to write a quick app to cut ogg/theora+vorbis files without reencoding, but as it turns out there is a need to write many things so I keep putting this one off. I'll get to it soon, this week perhaps.

Regarding the event itself, it's a bit strange. That is all I will say at the moment.

Also, I just got back yesterday from a trip out to Santander and Salamanca. I was visiting my sister first, who should be safely back in the States at this point. On the move in the Wingo family.

Salamanca was next, a city close to my heart. I studied there a year, starting in 1999, but had not been back in five years. In this return I had the pleasure of going with a friend of mine from that time, also estadounidense. We have determined that everything remains in its place. Well some things change, but man, I go in Birdland to have a beer in one of their bay-window tables overlooking Plaza España, and the same fellow is tending bar. Very comforting and home-like.

**hacks**

[Turn evolution IMAP caches into maildirs](//wingolog.org/pub/evo-cache-to-maildir.sh). God forbid you have to use this. Unrelated: evolution has been unpleasant in breezy. I would switch if I knew of a decent alternative. For personal definitions of decent.

[Stop libtool from checking for 17 different flavors of fortran](http://cvs.freedesktop.org/gstreamer/common/m4/as-libtool-tags.m4?rev=1.2&view=markup). Drop that macro somewhere and [invoke it](http://cvs.freedesktop.org/gstreamer/gstreamer/configure.ac?r1=1.373&r2=1.374). Although I was once partial to fortran I am no longer who I was.

**powerbook**

I took neither the red pill nor the blue pill but decided to bite apple's shiny metal ass, in the form of a beautiful aluminum powerbook. [People more eloquent than me attempt to explain why.](http://www.terrasoftsolutions.com/products/apple/laptops-desktops.shtml) I will say that I am very happy with the decision, although I reserve the right to curse Apple, ppc haters, and other entities conspiring to cause me problems.

[GDB 6.3 is worthless for threaded PPC apps](http://bugzilla.ubuntu.com/show_bug.cgi?id=4465). Solution: compile 6.1.1 from source. 6.2 might work, haven't tried it. Better solution, support DWARF2 on ppc.

I've fixed a few things in GStreamer 0.9 related to endianness.

Switching caps lock and control into their rightful places is crucial for me, but the apple caps lock hardware conspires against usability. Thankfully Hans Fugal whipped up a patch to allow you to [swap caps lock and control on an ibook or powerbook](http://hans.fugal.net/yodl/blosxom.cgi/mac/caps.html). Apply to the kernel sources, enable the configure option, and there you are. I applied to ubuntu's 2.6.12, but that's me. You can find a wholly-unsupported kernel for ubuntu breezy [here](//wingolog.org/pub/kernel-image-2.6.12_wingo.1.0_powerpc.deb).

Which brings us to the wireless. These laptops have the Airport Extreme, powered by the dreaded Broadcom chipset. Until a couple of weeks ago it was impossible to get this chipset to work under linux. Until [this guy](http://forums.gentoo.org/viewtopic-t-365647.html). Basically, you run [Mac-on-Linux](http://www.maconlinux.org/), share your wireless connection to a fake interface, and tunnel that connection into Linux. Works pretty well.

This was all based on the work from [Mattias Nissler](http://www-user.rhrk.uni-kl.de/~nissler/mol/). I did manage to get this working, with a bit of help from Zaheer Merali. I couldn't find any official distro of MOL to which the patches would apply, so I had to get him to tar up the gentoo sources, which already have the patches. I can put them somewhere if there is interest. Then when building you enable the pciproxy code and disable the tap and sheep network devices (tun should be on). And I did have to use Mattias' bootx.

Yes, architecturally it is nasty but I am getting tired so will stop musing about that.

Also I had to compile the [appletouch](http://www.popies.net/atp/) driver for the touchpad to work. Fortunately it is an easy compile, and it will be in Linux 2.6.13. Now to get the capslock patch upstream and I will be happy.

A quick list of non-working items: gl accelleration (nvidia sucks), brightness control (must be an nv framebuffer issue), external display, mol on a console, suspend/sleep. Will rant later about these.

## related articles

- [come on feet](/archives/2006/05/13/come-on-feet)
- [american dispatches](/archives/2006/03/04/american-dispatches)
- [quickly](/archives/2006/02/17/quickly)
- [enjoying the widening gyre](/archives/2006/01/23/enjoying-the-widening-gyre)
- [bang bang](/archives/2006/01/20/bang-bang)
- [pos-pavo](/archives/2005/12/08/pos-pavo)

Comments are closed.
