---
title: 'research!rsc: Division via Multiplication'
url: https://research.swtch.com/divmult
published: "2008-01-14T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/divmult
---

# research!rsc: Division via Multiplication

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Division via Multiplication Russ Cox January 14, 2008 *research.swtch.com/divmult* Posted on Monday, January 14, 2008.

Division is the most expensive integer operation you can ask of your CPU. A well-known optimization rewrites division by constant powers of two as bit shifts: if `x` is an unsigned integer, then `x>>8` is a fast way to compute `x/256`.

Division by constants that aren't powers of two are a little harder. Jim Blinn dedicated his November 1995 column (“ [Three Wrongs Make a Right](http://doi.ieeecomputersociety.org/10.1109/38.469535)”; subscription required) in IEEE Computer Graphics and Applications to tricks for computing `x/255` for 16-bit `x` (a common computation during alpha compositing). Since 1/255 is approximately the same as 257/65536, you can get pretty close to x/255 by computing `x*257/65536` (aka `(x*257)>>16)`). 257/65536 is a tiny bit bigger smaller than 1/255, so the approximation is sometimes too small by 1. Adding 257/65536 to nudge the values slightly upward results in a perfect approximation: for unsigned 16-bit x, x/255 = `(x*257+257)>>16`. No division!

In 1991, Robert Alverson described how to determine these approximations for the general case of arbitrary integer constants (“ [Integer Division Using Reciprocals](http://citeseer.ist.psu.edu/107754.html)”). Alverson was on the team building the 64-bit Tera Computer; they used the technique as the hardware implementation of integer division.

The Tera Computer didn't exactly catch on, but a few years later, Torbjörn Granlund and Peter Montgomery, inspired by Alverson's paper, implemented the technique in the GNU C compiler (gcc) for division by integer constants (“ [Division by Invariant Integers Using Multiplication](http://citeseer.ist.psu.edu/granlund94division.html)”). Gcc still uses this technique, as do other compilers. Gcc rewrites a 32-bit unsigned `x/255` into `(x*0x80808081)>>39` (the intermediate values are 64-bit integers).

Henry S. Warren's delightfully bit-twiddly book [Hacker's Delight](http://www.hackersdelight.org/) devotes 46 pages to this topic. His web site includes a [magic number computer](http://www.hackersdelight.org/magic.htm).

(Comments originally posted via Blogger.)

- [Taj](http://www.blogger.com/profile/17865728042714690702)(January 14, 2008 11:24 AM) s/tiny bit bigger/tiny bit smaller/ ?

- [Russ Cox](http://swtch.com/~rsc/)(January 14, 2008 11:54 AM) Fixed, thanks Taj.

- [knowak](http://knowak.wordpress.com/)(August 17, 2009 11:34 AM) Very insightful and a good starting point to explore the topic further. Thanks!

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(July 4, 2010 6:01 AM) And on some architectures, the resulting x \* 257 step can be done cheaply with an add and a barrel-shifter, e.g. ARM's "add r0, r0, r0 lsl #8", r0 = r0 + (r0 << 8), avoiding a multiply.

- Anonymous (January 20, 2011 5:07 AM) 66413.....25989
