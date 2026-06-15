---
title: 'research!rsc: The MOS 6502 and the Best Layout Guy in the World'
url: https://research.swtch.com/6502
published: "2011-01-03T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/6502
---

# research!rsc: The MOS 6502 and the Best Layout Guy in the World

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# The MOS 6502 and the Best Layout Guy in the World Russ Cox January 3, 2011 *research.swtch.com/6502* Posted on Monday, January 3, 2011.

The [MOS 6502](http://en.wikipedia.org/wiki/MOS_Technology_6502) was ubiquitous in its day. The 6502 and its slight variants were at the heart of the Apple II, the Atari 2600, the BBC Micro, the Commodore 64, and the Nintendo Entertainment System, among others. It's amazing to think that all five—each a very influential system in its own right—were all built around the same chip.

The 6502 was the brainchild of [Chuck Peddle](http://en.wikipedia.org/wiki/Chuck_Peddle), at the time an engineer with Motorola. Peddle was one of the engineers who worked on the [Motorola 6800](http://en.wikipedia.org/wiki/Motorola_6800), and one of his jobs was to, well, peddle the 6800 to customers. The customers loved everything about it except its $300 price tag. Peddle tried to convince management at Motorola to create a lower-cost microprocessor, but Motorola didn't want to have such a chip cut into their not insubstantial profits from the 6800, and they told him in no uncertain terms that they wouldn't build such a chip. In response, Peddle and a handful of other 6800 engineers left Motorola and built one themselves. It was the MOS Technology 6502 and sold for $25. Even though both the 6800 and 6502 had a clock rate of 1 MHz, the 6502 had a minimal instruction pipeline that overlapped the fetch of the next instruction with the execution of the current one when possible, giving it a significant performance boost. And of course it sold for ten times less. So it ended up everywhere.

The story of the 6502 makes up the first chapter of Brian Bagnall's *[On the Edge: the Spectacular Rise\_ _and Fall of Commodore](http://books.google.com/books?id=17BFAAAACAAJ)*. My favorite part of the description of the development of the 6502 is the actual chip layout. These days, you can't design and lay out a computer chip without a computer. An Intel Core 2 chip has hundreds of millions of transistors. The 6502 had 3,510, and an engineer—a person, not a computer—had to draw each one by hand to lay out the chip. Mainly it was a single engineer, [Bill Mensch](http://en.wikipedia.org/wiki/Bill_Mensch).

But it gets better. Once the layout was completed and double-checked—a process that meant months using a ruler!—it still had to be converted into a Rubylith [photomask](http://en.wikipedia.org/wiki/Photomask) that would etch the right patterns onto the silicon. The photomask for the 6502 was the size of a large table—large enough that the engineers crawled around on top of it to perform the job of cutting the layout out of the mask, all the while being careful to wear clean socks with no holes, so that stray toenails didn't insert traces in the mask where they didn't belong.

The most amazing part about the whole process is that they got the 6502 right in one try. Quoting *On the Edge*:

> Bil Herd summarizes the situation. “No chip worked the first time,” he states emphatically. “No chip. It took seven or nine revs \[revisions\], or if someone was real good they would get it in five or six.”
>
> Normally, a large number of flaws originate from the layout design. After all, there are six layers (and six masks) that have to align with each other perfectly. Imagine designing a town with every conceivable layer of infrastructure placed one on top of another. Plumbing is the lowest layer, followed by the subway system, underground walkways, buildings, overhead walkways, and finally telephone wires. These different layers have to connect with each other perfectly; otherwise, the town will not function. The massive complexity of such a system makes it likely that human errors will creep into the design.
>
> After fabricating a run of chips and probing them, the layout engineers usually have to make changes to their original design and the process repeats from the Rubylith down. “Each run is a couple of hundred thousand \[dollars\],” says Herd.
>
> Implausibly, the engineers detected no errors in \[Bill\] Mensch's layout. “He built seven different chips without ever having an error,” says Peddle with disbelief in his voice. “Almost all done by hand. When I tell people that, they don't believe me, but it's true. This guy is a unique person. He is the best layout guy in the world.”

The first chapter of *On the Edge* is posted on [Bagnall's web site](http://www.variantpress.com/view.php?content=toc) in [HTML](http://www.variantpress.com/view.php?content=ch001) or [PDF](http://www.variantpress.com/contents/ch001%20MOS%20Technology.pdf). The chapter says that Peddle “created a concept called *pipelining*,” which could be interpreted as saying that the 6502 was the first pipelined processor and that Peddle invented it. Does anyone know? \[Note: See update at end of post.\]

Fast forward thirty years. Computers are now old enough that there can be significant interest in maintaining the history of these old, venerable designs. The actual paper designs of the 6502 are long gone.

A team of three people—Greg James, Barry Silverman, and Brian Silverman—accumulated a bunch of 6502 chips, applied sulfuric acid to them to strip the casing and expose the actual chips, used a high-resolution photomicroscope to scan the chips, applied computer graphics techniques to build a vector representation of the chip, and finally derived from the vector form what amounts to the circuit diagram of the chip: a list of all 3,510 transistors with inputs, outputs, and what they're connected to. Combining that with a fairly generic (and, as these things go, trivial) “transistor circuit” simulator written in JavaScript and some HTML5 goodness, they created an animated 6502 web page that lets you [watch the voltages race around the chip
as it executes](http://www.visual6502.org/JSSim/index.html). For more, see their web site [visual6502.org](http://www.visual6502.org/).

Oh, and it actually works. They applied the same technique to build the transistor map for an Atari 10444D TIA chip, which connected the 6502 to the television in the original Atari 2600, and then they simulated both chips together and were [able to run actual Atari 2600 games](http://www.visual6502.org/docs/6502_in_action_14_web.pdf). So the transistor map is probably (very close to) correct. More impressively, they got to that point *without debugging*. Their SIGGRAPH 2010 [abstract](http://www.visual6502.org/docs/6502_siggraph2010_abs.pdf) explains that there were only 8 errors in the map of 20,000 components, and all the errors were spotted during the vectorization process. History repeats itself.

[Michael Steil](http://www.michael-steil.de) took the circuit information and started looking closely at how the chip did what it did. At last week's Chaos Communication Congress 27C3 conference, he gave a 50-minute talk that introduces the 6502 architecture and then uses the circuit diagram to explain various details and undocumented features of the chip. The original announcement is at [Steil's blog](http://www.pagetable.com/?p=512). There is a version of the [talk on YouTube](http://www.youtube.com/watch?v=HW9AWBFH1sA), and in the blog comments you'll find links to higher-resolution copies. It's well worth watching, in any form, and it's fun to see how much Peddle, Mensch, and the rest of the 6502 team packed into that tiny number of transistors.

**Update, January 2015.**

The links above to Chapter 1 of *On the Edge* on the publisher site no longer work, but there is a [copy of the HTML version at the Internet Archive](https://web.archive.org/web/20120721114927/http://www.variantpress.com/view.php?content=ch001).

As noted in the comments below, the [Manchester Atlas](http://en.wikipedia.org/wiki/Atlas_Computer_%28Manchester%29), the [CDC 6600](http://en.wikipedia.org/wiki/CDC_6600), and the [IBM 7030 (Stretch)](http://en.wikipedia.org/wiki/IBM_7030_Stretch) all used pipelining at least a decade before the 6502 was created. Also, what the 6502 did was not really pipelining in the modern sense. It overlapped instruction execution with the fetch of the next instruction in some cases, as did the Intel 8080 (which beat the 6502 to market) and other microprocessors of its era. Bagnall's claim that Peddle created pipelining for the 6502 seems to be completely without justification.

Also, in the linked chapter, Bagnall explains that “As Motorola publicly unveiled the 6800, Chuck Peddle and seven coworkers from the engineering and marketing department left Motorola to pursue their own vision. The team included Will Mathis, Bill Mensch, Rod Orgill, Ray Hirt, Harry Bawcum \[sic\], Mike James \[sic\], Terry Holt, and Chuck Peddle.” Later, Bagnall writes, “The layout team consisted of two main engineers: Bill Mensch and Rod Orgill. A third engineer, Harry Bawcum \[sic\], aided the layout artists. It was their task to turn an abstract block diagram into a large-scale representation of the surface of the microprocessor. Orgill was responsible for the 6501 chip, Mensch the 6502.” There is little mention of others again. Mensch gets all the credit. In January 2015, I received the following note via email from Harry Bawcom:

> Just an FYI.
>
> Bill Mensch did not layout the 6502, I did, with the help of two others.
>
> In truth we all worked as a team but Bill didn't draw a single line on the first version of the 6502. My initials were on the die.
>
> I wasn't there for the later versions.
>
> Interesting. There were two other circuit designers involved, one senior to Bill. They deserve credit, too.

An [EE Times article from August 1975](http://www.commodore.ca/gallery/magazines/misc/mos_605x_team_eetimes_august_1975.pdf) shows a picture of the 6502 team, along with a caption listing Harry Bawcom as a “layout designer” and Bill Mensch as a “design engineer,” which seems to corroborate Bawcom's note. (Note that the photo printed in the article has been mirror-image reversed.) I was right to be skeptical of the book's claim that Chuck Peddle and the 6502 introduced pipelining. I was probably wrong not to be skeptical of the rest of the story I quoted. It was too good to be true. The Visual 6502 story remains a nice ending, however.

P. S. If you liked this, you might also like [Computing Machines](http://research.swtch.com/2010/04/computing-machines.html). Also, Barry Silverman and Brian Silverman (mentioned above) were two of the people behind the [PDP-1 simulator
written in Java that runs Spacewar!](http://spacewar.oversigma.com/)

(Comments originally posted via Blogger.)

- [Jan-willem](http://www.blogger.com/profile/12548850641640358518)(January 3, 2011 8:16 AM) Wikipedia seems to think [pipelining](http://en.wikipedia.org/wiki/Instruction_pipeline) was used by Cray in the XMP. And I'm pretty sure it goes back at least as far as the CDC days, and probably to IBM. But it wouldn't surprise me to learn that the 6502 was the first pipelined microprocessor.

- [Russ Cox](http://swtch.com/~rsc/)(January 3, 2011 8:36 AM) @jan-willem: Thanks. I saw that but couldn't reconstruct a timeline. The 6502 shipped in 1975, the same year that the Cray-1 did. The Cray X-MP was announced in 1982.

- [Preston](http://www.blogger.com/profile/13654427570630206482)(January 3, 2011 10:23 AM) I thought the IBM Stretch was the first pipelined processor (delivered in '61), but Wikipedia notes that the Illiac II may have been the first pipelined processor. And that the Z1 (circa '39) had a simple form of pipelining.

Preston

- Anonymous (January 3, 2011 12:42 PM) Atari 2600 didn't use the 6502 ;)

- [Russ Cox](http://swtch.com/~rsc/)(January 3, 2011 12:55 PM) @Anonymous: The 6507 was a 6502 with some address lines cut off. It was one of the "slight variants".

The NES was a similar story: it used a Ricoh 2A03, which was a 6502 and some other video logic on the same core.

- [John Lazzaro](http://www.blogger.com/profile/11265093304899746444)(January 3, 2011 1:54 PM)This post has been removed by the author.

- Anonymous (January 3, 2011 1:55 PM) The RICOH (NES) processor was not strictly a 6502, in that it omitted decimal mode. The official story is that it was left out "for space reasons", but it is far more likely that the omission had something to do with that being the main patent that MOS Technology had on the chip. :-)

- [Russ Cox](http://swtch.com/~rsc/)(January 3, 2011 2:05 PM) @Anonymous: Are you saying that by cutting the decimal mode part they were able to drop in an otherwise unmodified 6502 circuit on the silicon for the 2A03 without having to pay royalties to MOS Technology? That seems surprising to me.

In this video http://www.youtube.com/watch?v=N9DYmlprCKA at 9:30 you can see a die shot of the 2A03 and the 6502 is visible in the bottom right.

- [Russ Cox](http://swtch.com/~rsc/)(January 3, 2011 6:49 PM) From [Howard\_Beale on reddit](http://www.reddit.com/r/programming/comments/evnom/the_mos_6502_and_the_best_layout_guy_in_the_world/c1bc5vy):

Jeri Ellsworth did a really great (long) interview with Chuck last year. It is well worth the watch.

Part 1: [http://www.youtube.com/watch?v=cns75TIrzb8](http://www.youtube.com/watch?v=cns75TIrzb8)

- Anonymous (January 4, 2011 3:08 AM) The people saying that pipelining was used by Cray and CDC are right. It was in the Control Data 6600, released in 1964, a machine which pioneered the next 40 years of CPU technology. It is worth hunting down a copy of Jim Thornton's book *Design of a Computer* for the full story.

- Anonymous (January 4, 2011 3:44 AM) 6502 cores ended up in winmodems too. And the design evolved into the ARM processor of today. A venerable ancestor indeed.

- [Leo Richard Comerford](http://www.blogger.com/profile/13824237190916037661)(January 4, 2011 6:05 AM) It seems that as CEO of the [Western Design Centre](http://www.westerndesigncenter.com/), Mensch is still selling 6502-compatible chips and "the 65xx IP". So whether or not Mensch or the WDC still has the original 6502 documents, it seems that the reason the internal workings of the 6502 remained unclear before the Visual 6502 project isn't because the knowledge had been lost in the mists of time, it's because that knowledge was (and is) still regarded as commercially valuable by the people who had it.

- [Leo Richard Comerford](http://www.blogger.com/profile/13824237190916037661)(January 4, 2011 6:48 AM) Anonymous:

I haven't read it myself, but I note that Thornton's [*Design of a Computer*](http://www.bitsavers.org/pdf/cdc/cyber/books/DesignOfAComputer_CDC6600.pdf) has been scanned and uploaeded to bitsavers.org.

- [Frederick Polgardy](http://www.blogger.com/profile/07838643783546960230)(January 4, 2011 6:54 AM) Not the Atari 2600, the Atari 8-bit computers: 400/800, 600/800/1200XL.

- [shaurz](http://www.blogger.com/profile/03588038254545671774)(January 4, 2011 7:37 AM) It didn't evolve into the ARM processor, they are completely unrelated apart from Acorn using the 6502 in their early microcomputers before they designed the ARM.

- Anonymous (January 4, 2011 9:12 PM) I'd previously read that the 6800 and 6502 were essentially pin compatible. Your article leads to fresh insights as to how and why that was the case.

- [Russ Cox](http://swtch.com/~rsc/)(January 4, 2011 9:43 PM) The 6800 and the 6501 were pin-compatible - same size chip, same placement for power, data, address lines - but I don't believe that means instruction-set compatible.

The 6502 was not pin-compatible.

http://www.variantpress.com/view.php?content=ch001

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(January 5, 2011 10:08 AM) shaurz is correct, the CISC 6502 didn't evolve into the ARM. Acorn employee [Sophie Wilson](http://www.sophie.org.uk/) who was expert at 6502 and author of the 16KiB ROM of 6502 code containing BBC BASIC, designed the ARM's RISC instruction set and went on to write BBC BASIC V in ARM along with novel, for their time, video codecs.

- Anonymous (January 5, 2011 10:10 AM) Bill Mensch designed the 65816 microprocessor too, that's the chip used in the Super Nintendo and Apple IIgs. He says that all the design and layout was done by himself and his sister.

- [Chad](http://www.blogger.com/profile/16659358584605396355)(January 5, 2011 8:59 PM) To be fair, the deisgn was not "right the first time". There was at least one erratum, in the ROR instruction, as described [here](http://www.pagetable.com/?p=406).

- Anonymous (January 6, 2011 8:09 PM) I'm amazed that they had to resort to reverse engineering such a thing, that there were no actual records or circuit diagrams left in an archive somewhere. Considering the impact the chip had, the circuit diagrams should have been preserved with the same reverence as the Constitution.

It makes one wonder what other things we've lost along the way.

- Anonymous (January 6, 2011 8:44 PM) The first pipelined machine was the Manchester Atlas in 1962.

http://en.wikipedia.org/wiki/Atlas\_Computer\_%28Manchester%29

- Anonymous (January 6, 2011 9:33 PM) C= 64 used a 6510 not a 6502.

http://en.wikipedia.org/wiki/MOS\_Technology\_6510

- Anonymous (January 6, 2011 10:11 PM) >Anonymous Anonymous said...



\> C= 64 used a 6510 not a 6502.



Wasn't it the Vic20?

- Anonymous (January 6, 2011 11:54 PM) The 6510 is just a 6502 with a tacked on 8-bit data port at $00/$01.

Commodore bought MOS Technology (Chuck Peddle's company) when they got into computers so they could build all their chips in-house.

- [Mikael Tillenius](http://www.blogger.com/profile/15958313282621563010)(January 7, 2011 6:07 AM) The Intel 8080 (introduced 1974, one year before the 6502) used similar pipelining. Btw, this is not pipelining in the modern sense. The processors just overlapped the last part of one instruction with the fetching of the next one. This was a well known technique at the time.

- [Mike Gora](http://www.blogger.com/profile/04402667224687572273)(January 7, 2011 8:04 PM) By the mid-1970s at the latest, Fortran compilers were taking advantage of pipelining, and were even optimized for different depths of pipelining on different versions of a mainframe. (it was just before that, around 1973, that virtual memory became commonplace, and Fortran code had to be optimized to avoid the worst-case paging that would occur if a two-dimensional array were initialized looping through the wrong subscript first.)

- Anonymous (January 8, 2011 10:01 AM) So what about the 65c02 that was used for Apple //e i beleive...

- Anonymous (January 9, 2011 6:42 PM) I had one of the early 6502's with the defective ROR instruction which Chad mentioned. And I had the MOS programming manual which omitted the instruction. (Wish I still had both). The part was in a ceramic package. Once, on a lark, I replaced the 6502 in a friend's Commodore PET with my ROR-free part. The PET started to run, but got progressively stranger until it eventually crashed after several minutes of oddities on the screen.
