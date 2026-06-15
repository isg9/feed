---
title: 'research!rsc: Permissive Access Links'
url: https://research.swtch.com/pal
published: "2008-02-25T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/pal
---

# research!rsc: Permissive Access Links

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Permissive Access Links Russ Cox February 25, 2008 *research.swtch.com/pal* Posted on Monday, February 25, 2008.

In 2004, Steve Bellovin gave a talk at Usenix Security speculating about permissive access links (PALs), the (supposedly impossible to bypass) locks that protect nuclear weapons. He repeated the talk in 2006 at the general Usenix. In Bellovin's talk, “ **[Nuclear Weapons, Permissive Action Links, and the History of Public Key Cryptography](http://www.usenix.org/events/usenix06/tech/mp3/bellovin.mp3)**” (MP3; also [PDF](http://www.usenix.org/events/usenix06/tech/slides/bellovin_2006.pdf) and [HTML](http://64.233.169.104/search?q=cache:_gevj9vbdqsJ:www.usenix.org/events/usenix06/tech/slides/bellovin_2006.pdf)), he says that “Bypassing a PAL should be, as one weapons designer graphically put it, about as complex as performing a tonsillectomy while entering the patient from the wrong end.” But how do they work? Are there lessons that apply to building other kinds of secure systems? He touches on these questions, but in the end, it's mostly speculation. Even so, it's a fascinating talk.

He does tease out a few interesting [historical details](http://www.cs.columbia.edu/~smb/nsam-160/). In particular, [National Security Action Memorandum 160](http://www.cs.columbia.edu/~smb/nsam-160/pg1.html), signed by President Kennedy, has been claimed by former NSA insiders to be the impetus for the NSA's invention of public key cryptography. There is no evidence that public key cryptography ended up being used in PALs, but it's possible that digital signatures were invented in direct response to the requirement that, after a weapon was launched, it be possible to determine who authorized the launch. It's also possible that public key cryptography was invented and used to transmit the PAL codes securely.

Other interesting facts. The U.S. offered PALs to the Soviets (presumably to keep weapons from falling into other hands), but they turned them down. For years after the initial U.S. PAL deployments, the launch codes were all set to 00000000. The bandwidth of the extra-long frequency [extremely low frequency](http://en.wikipedia.org/wiki/Extremely_low_frequency) (ELF) communication link to submerged submarines is 1 bit/minute.

(Comments originally posted via Blogger.)

- [Monstre](http://www.blogger.com/profile/03975843196252061184)(August 8, 2009 11:00 AM) hi .. !! nice blog .. visit mine .. www.spyfree.info

i have added your .. expecting the same
