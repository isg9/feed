---
title: The Factory Floor, Part 3 of 4:<br>Industrial Design for Startups
url: https://www.bunniestudios.com/blog/2013/the-factory-floor-part-3-of-4industrial-design-for-upstarts/
published: "2013-01-19T10:49:38Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=2790
---

# The Factory Floor, Part 3 of 4:<br>Industrial Design for Startups

The geek tour continues on. Akiba has new posts up covering our visit to a [motor factory](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-16-Lotus-Motors.html), [Huawei](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-14-Huawei.html), [CTS](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-14.html), and also a side trip to get [full custom clothes and bags tailor made](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-12-Fabric-Mall.html). The photos from the [motor factory](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-16-Lotus-Motors.html) and the [custom tailoring expedition](http://www.freaklabs.org/index.php/Blog/MIT-Media-Lab-Shenzhen-2013/MIT-Media-Lab-Shenzhen-2013-01-12-Fabric-Mall.html) came out particularly well.

And now, on to part 3 of 4 of “The Factory Floor” series…

**Industrial Design for Startups:**

**Guerrilla Engineering on a Shoestring Budget**

Sex sells. The performance of a CPU or amount of RAM in a box, to within a factor of two or so, is less important to a typical consumer than how the device looks. Apple devices command a hefty premium in part because of their slick industrial design, and many product designers aim to emulate the success of Sir Jony Ives in their own products.

There are many schools of thought in industrial design. One school invokes the monastic designer, coming up with a beautiful, pure concept, and the only thing the production engineers can do is spoil the purity of the design. Another school invokes the pragmatist designer, working closely with the production engineers, hammering out gritty compromises to produce an inexpensive and high-yielding design.

In my experience, neither extreme is compelling. The monastic approach often results in an unmanufacturable product that is either late to market or exorbitant to produce. The pragmatist approach often results in in a cheap look and feel, to which consumers have trouble assigning a significant value. The real trick is understanding how to strike a balance between the two.

Trim and finish are difficult, and therefore a point of distinction when it comes to design. The current design fad is minimalism, with an emphasis on “honest” finishes. An honest finish features the natural properties of the material systems in play, and eschews the use of paints and decals. Minimalist, honest designs are very hard to manufacture. Minimal designs have…well, minimal, features – and as a result even tiny blemishes stand out. Honest finishes likewise can be very difficult, as an honest finish means no paint: all the burs, gates, sinks, knits, scoring and flow lines that are a fact of life in manufacturing are laid naked before the consumer. As a result, this school of design requires well-made tools that are constantly checked and maintained throughout production.

If you don’t have pockets deep enough to invest in new equipment and capabilities on behalf of your factory (i.e., if you’re not a Fortune 500 company), the first step is to learn the vocabulary available. A design vocabulary is defined by the capabilities of the factory or factories producing the goods. What materials, what finish, what tolerances are achievable, what fastening technology is available – these are all heavily dependent upon the processes available.

Therefore, I find that visiting a factory in person early in the design process results in a better design result. In a factory visit, some design vocabulary will be discarded, but some new vocabulary will be discovered as well – the engineers who work the factory day in and day out develop process innovations that can open up novel design possibilities that are not knowable without the on-site visit.

The [chumby One](http://www.bunniestudios.com/blog/?p=611) contains a concrete example of the impact manufacturing process can have on design outcome. In the original concept art, the blue highlight around the front edge was added to evoke the feeling of a speech balloon, like those used in captioning comics – the idea being the chumby is captioning your world with snippets from the Internet.

![](http://bunniestudios.com/blog/images/verizon_3g_c1_cnxn.jpg)

It turns out the implementation of such a blue trim across a raised surface is very hard. At the first factory, we implemented the highlight using paint. Silk screening was not an option because the shape wasn’t flat enough. Pad printing can handle curved surfaces, but the alignment wasn’t good enough, as the tiniest bleed over the edge looked terrible from the side. Decals and stickers likewise could not achieve the alignment required. In the end, a small channel had to be carved to contain the paint, and a stencil plus spray paint process was employed to create the highlight. The yield was terrible – in some lots, over 40% of the cases were being thrown away due to painting errors. Fortunately, plastic is cheap, so throwing away every other case after painting had a net cost impact of about $0.35.

![](http://bunniestudios.com/blog/images/cr-c1-paint1-note.png)

![](http://bunniestudios.com/blog/images/cr-c1-paint2-note.png)

Mid-way through production, we migrated to a second source facility. They had a different plastic molding capability, and unlike the first factory, the second facility could do double-shot molds. Double-shot molds have twice the number of tools, but they can injection mold two different colors, or even different materials, into the same mold. Thus, at the new factory, we opted to use a double-shot process for the thin blue strip, instead of painting. The results were stunning. Every unit came off the line with a sharp, crisp blue line; and no paint meant a more honest, clean finish. However, the cost per case jumped to $0.94 a piece due to the more expensive process, despite the 100% yield. In fact, it would be cheaper to throw away more than half of the painted cases, but even the best painted cases could not compare to the quality of the finish delivered by the double-shot tool.

![](http://bunniestudios.com/blog/images/cr-c1-doubleshot.png)

Another great example of how tweaking a factory process can improve a product’s appearance is the Arduino motherboard. The wonderfully detailed artwork on the back side, sporting an outline of Italy and very fine lettering, isn’t silkscreen. They actually put on two layers of soldermask, one blue, and one white. Because soldermask is applied using a photolithographic process, the resolution, consistency and alignment of the artwork is much better than a silkscreen. And since an Arduino’s look is the circuit board, it gives the product a distinctive high-quality look that is difficult to copy using conventional processing methods.

[![](http://bunniestudios.com/blog/images/cr-arduino-sm.jpg)](http://bunniestudios.com/blog/images/cr-arduino.jpg)

Thus, the process capability of the factory – painting vs. double-shot molding, double soldermasking vs. silkscreening – can have a real effect in the outcome of a product’s perceived quality, without a huge impact on cost. However, a factory may not appreciate the full potential of their processes, and so it requires a designer’s direct interaction to realize the potential. Unfortunately, many designers don’t visit a factory until something has gone wrong, at which point the tools are cut and even if they see a cool process that could solve all their problems, it’s often too late.

Design is an intensely personal activity, and as a result every designer will develop their own process. This is the general process I might use to develop a product on a tight, startup budget:

1\. Every design starts with a sketchbook. First, decide on the soul and identity of the design, and pick a material system and vocabulary that suits your concept. But don’t fall in love with it…

2\. Break the design down by material system, and identify a factory capable of producing each material system.

3\. Visit the facility, and take note of what is actually running down the production lines. Don’t get too drawn in by the sample room or one-off bits. Practice makes perfect, and from the operators to the engineers they will do a better job of executing things they are doing on a daily basis than reaching deep and exercising an arcane capability.

4\. Re-evaluate the design based on a new understanding of what’s possible, and iterate. This may require going back to step 1, or it may just require small tweaks. But this is the stage at which it’s easiest to make compromises without sacrificing the purity of the design.

5\. Rough out the details of the design – pick parting lines, sliding surfaces, finishes, fastening systems, etc. based upon what the factory can do best.

6\. Pass a revised drawing to the factory, and work with them to finalize details such as draft angles, fastening surfaces, internal ribbing, etc.

7\. Validate the design using a 3D print and extensive 3D model checks.

8\. Identify features prone to tolerance errors, and trim the initial tool so that the tolerance favors “tool-safe” modifications. For example, in injection molding it is easier to remove steel than to add it to a tool, so target the initial test shot to have less plastic than too much on critical dimensions. A button is an example of a mechanism that benefits from tuning: it’s hard to predict from CAD or 3D prints exactly how a button will feel, and getting that tactile feel just perfect usually requires a little trimming of the tool.
