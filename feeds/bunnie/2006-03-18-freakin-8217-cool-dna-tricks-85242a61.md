---
title: Freakin&#8217; Cool DNA Tricks
url: https://www.bunniestudios.com/blog/2006/freakin-cool-dna/
published: "2006-03-18T11:27:51Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=84
---

# Freakin&#8217; Cool DNA Tricks

So tonight I took in my copy of [Nature](http://www.nature.com) from the mail and opened it up…and saw smiley faces. I thought this was the coolest thing I had seen in a while: Paul W.K. Rothemund, a researcher at Caltech, has figured out a way to cause DNA to fold into arbitrary patterns (Nature calls it [DNA origami](http://www.nature.com/nature/journal/v440/n7082/index.html)). The patterns include, of all things, a nano-scale smiley face (I’m glad to see he has a sense of humor! The article actually has a number of other very interesting patterns that demonstrate the utter generality of the technique). And when I say nano-scale smiley, I mean a smiley face that’s smaller than 100 nm across. Given that fabs struggle to get a single rectangular strip of polysilicon to print that is 65 nm, this is a really remarkable result. Although I haven’t had a chance to fully digest the article (after all, I’m reading this the evening of St. Patty’s day), the technique used is ingenious. They use a well-characterized 7 kBase DNA sequence from the bacteriophage virus M13mp18 and they use a combination of computer scripts to determine a set of oligonucleotides (short DNA sequences) that they can easily synthesize to “staple” the structure into the desired pattern. As far as I can tell, they take a gob of these genomes and mix it with these oligos in a test tube, shake, and *viola*, they self-assemble into patterns–such as the smiley face below (pictures from the Nature article linked above–hope this is fair use…)

![](http://bunniestudios.com/blog/images/smileys.jpg)

![](http://bunniestudios.com/blog/images/onesmiley.jpg)

The scale bar on the top image is 100 nm(!). The mind boggles. Even though one can obviously see defectivity in the self-assembled arrays, they are so tiny that it sort of doesn’t matter. Five or six of these smiley faces easily fit onto a single small-sized transistor’s gate from one of the silicon die shown in the blog post below this one. And these are *complex* structures, not some simple rectangle of polysilicon. Fortunately, guys like Andre’ DeHon have thought about how to leverage such phenomenal small-scale integration through [fault-tolerance techniques](http://www.cs.caltech.edu/research/ic/abstracts/seven_strategies_ieeedt2005.html) that map well into the computer domain. Oh, and I forgot to mention, the author of the article notes that there are many forms of chemically modified DNA available that could be incorporated into these scaffords to enable the incorporation of even more wacky self-assembled features.

I think maybe it’s past time for me to move up the periodic table one row and get into Carbon hacking. The good work of guys like Rothemund, [Knight](http://en.wikipedia.org/wiki/Tom_Knight), and [Endy](http://openwetware.org/wiki/Endy_Lab) are bringing [synthetic biology](http://en.wikipedia.org/wiki/Synthetic_biology) to a level where even I might be able to play with it and create something neat, if not just artistic and fun.

After sleeping on the thought for a night, I realized one of the really important aspects of this work is that they used an *arbitrary* DNA ring as the substrate for these structures…M13mp18 happens to be well known, but I suppose any fully sequenced DNA strand might also work. It’d be funny to see smiley faces made with my Y chromosome (despite being my smallest, gimpiest, chromosome it’s still 23 million base pairs…maybe a bit too big for this technique). At any rate, the key is that you only need to synthesize a couple hundred short segments of DNA (a relatively easy task) to pin down a very long segment of sequenced DNA (something that is currently hard to sythesize) into an arbitrary shape.
