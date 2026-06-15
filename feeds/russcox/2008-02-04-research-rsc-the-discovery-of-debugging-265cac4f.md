---
title: 'research!rsc: The Discovery of Debugging'
url: https://research.swtch.com/discover-debug
published: "2008-02-04T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/discover-debug
---

# research!rsc: The Discovery of Debugging

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# The Discovery of Debugging Russ Cox February 4, 2008 *research.swtch.com/discover-debug* Posted on Monday, February 4, 2008.

Debugging was a surprise. When the early computer pioneers built the first programmable computers, they assumed that writing programs would be straightforward: think hard, write the program, done.

[Maurice Wilkes](http://awards.acm.org/citation.cfm?id=1001395&srt=all&aw=140&ao=AMTURING), creator of the EDSAC, the first stored-program computer, wrote what might be the [first programming textbook](http://mitpress.mit.edu/book-home.tcl?isbn=0262231182) in 1951, with David Wheeler (inventor of the subroutine call!) and Stanley Gill. It warns: “Experience has shown that such mistakes are much more difficult to avoid than might be expected. It is, in fact, rare for a program to work correctly the first time it is tried, and often several attempts must be made before all errors are eliminated.”

In [his memoir](http://mitpress.mit.edu/book-home.tcl?isbn=0262231220), Wilkes recalled the exact moment he realized the importance of debugging: “By June 1949, people had begun to realize that it was not so easy to get a program right as had at one time appeared. It was on one of my journeys between the EDSAC room and the punching equipment that the realization came over me with full force that a good part of the remainder of my life was going to be spent in finding errors in my own programs.”

Brian Hayes's excellent 1993 essay “ **[The Discovery of Debugging](http://bit-player.org/bph-publications/Sciences-1993-07-Hayes-EDSAC.pdf)**” (skip to page 32) describes the history in detail.

Martin Campbell-Kelly's 1992 essay “ [The Airy Tape: An Early Chapter in the History of Debugging](http://dx.doi.org/10.1109/85.194051)” (subscription required) gives more details on one of the stories in Hayes's article: the successful debugging, after 41 years, of the first hard bug. The bug? A single-precision floating-point value used in a double-precision context.

Hayes's essay itself contains a bug worth mentioning. It says that the first three programs run on the EDSAC ran correctly the first time, an understandable but incorrect inference from Campbell-Kelly's article. In fact the second program ever run, which merely printed primes, was buggy. The [EDSAC log](http://www.cl.cam.ac.uk/Relics/elog.html) for May 7, 1949 reads “Table of primes attempted — programme incorrect.” A correct primes program didn't run until two days later.

(Comments originally posted via Blogger.)

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(August 11, 2010 9:45 AM) Brian Hayes's essay " [The Discovery of Debugging](http://bit-player.org/bph-publications/Sciences-1993-07-Hayes-EDSAC.pdf)" has moved.
