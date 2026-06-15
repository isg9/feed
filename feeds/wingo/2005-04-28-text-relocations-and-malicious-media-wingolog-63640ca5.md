---
title: text relocations and malicious media — wingolog
url: https://wingolog.org/archives/2005/04/28/text-relocations-and-malicious-media
published: "2005-04-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/04/28/text-relocations-and-malicious-media
---

# text relocations and malicious media — wingolog

## [text relocations and malicious media](/archives/2005/04/28/text-relocations-and-malicious-media)

28 April 2005 7:59 PM

- [hacks](/tags/hacks)

I still can't figure out whether Ulrich Drepper's [DSO optimization paper](http://people.redhat.com/drepper/dsohowto.pdf) is just picking nits, showing the young whippershnappers their (our) place, or whether he really has a point.

In any case, he makes the case that text relocations, caused by non-PIC code in shared libraries, are bad, because they prevent those memory pages from being shared. Fine. But he goes on to mention some other aspects of text relocations as well:

> Generating DSOs so that text relocations are necessary (see section 2) means that the dynamic linker has to make memory pages, which are otherwise read-only, temporarily writable. The period in which the pages are writable is usually brief, only until all non-lazy relocations for the object are handled. But even this brief period could be exploited by an attacker. In a malicious attack code regions could be overwritten with code of the attacker's choice and the program will execute the code blindly if it reaches those addresses.
>
> During the program startup period this is not possible since there is no other thread available which could perform the attack while the pages are writable. The same is not true if later, when the program already executes normal code and might have start threads, some DSOs are loaded dynamically with dlopen. For this reason creating DSOs with text relocation means unnecessarily increasing the security problems of the system.

A quick look to see what libraries generate text relocations on my system:

```
$ for i in /usr/lib/*.so; do
    readelf -d $i 2>/dev/null | sed -e "s,^,$i: ," | grep TEXTREL
  done
/usr/lib/libdv.so:  0x00000016 (TEXTREL)            0x0
/usr/lib/libfftw3f.so:  0x00000016 (TEXTREL)        0x0
/usr/lib/libglide3.so:  0x00000016 (TEXTREL)        0x0
/usr/lib/libglide3x.so:  0x00000016 (TEXTREL)       0x0
/usr/lib/libHermes.so:  0x00000016 (TEXTREL)        0x0
/usr/lib/libmpeg2convert.so:  0x00000016 (TEXTREL)  0x0
/usr/lib/libmpeg2.so:  0x00000016 (TEXTREL)         0x0

```

Sounds like exactly the situation with GStreamer (or other media processors): take an untrusted data source, run it through your code, based on the type load up some dynamic libraries, then run. And it just so happens that most of the libraries that have textrels are the kinds that would be dynamically loaded to process untrusted data. Multiple beers for the first one to make an exploit on this, I'd like to see it. A bit trickier than a simple stack smashing.

**with apologies to [mr. guthrie](http://www.arlo.net/lyrics/alices.shtml)**

> They got a building down New York City, it's called Whitehall Street, where you walk in, you get injected, inspected, detected, infected, neglected and selected. I went down to get my physical examination one day, and I walked in, I sat down, got good and drunk the night before, so I looked and felt my best when I went in that morning. \`Cause I wanted to look like the all-American kid from New York City, man I wanted, I wanted to feel like the all-, I wanted to be the all American kid from New York, and I walked in, sat down, I was hung down, brung down, hung up, and all kinds o' mean nasty ugly things. And I waked in and sat down and they gave me a piece of paper, said, "Kid, see the psychiatrist, room 604."

Criminal record check, check. Medical check, check. Two 3" by 2" color glossy photos, sans circles and arrows, check. Watch out spanish embassy here I come.

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [steal our code](/archives/2007/05/03/steal-our-code)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)
- [cvs-bisect](/archives/2006/01/20/cvs-bisect)

Comments are closed.
