---
title: (Well Executed) Counterfeit Chips
url: https://www.bunniestudios.com/blog/2007/well-executed-counterfeit-chips/
published: "2007-10-17T08:52:11Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=208
---

# (Well Executed) Counterfeit Chips

Below are two chip specimen, purchased from an Asian source, that were recently called to my attention. I borrowed them to write this blog post.

[![](http://bunniestudios.com/blog/images/fakest_chips_sm.jpg)](http://bunniestudios.com/blog/images/fakest_chips.jpg)

The chips claim to be ST19CF68’s, a “CMOS MCU Based Safeguard Smartcard I/O with Modular Arithmetic Processor”. It seems these chips are normally sold in smart-card or diced wafer format, but curiously, these are SOIC-20 packaged devices.

The top chip in the pair has its epoxy top dissolved, and this is what it contains:

![](http://bunniestudios.com/blog/images/fakest_dieshot.jpg)

Kind of a small die for such a complex MCU, especially in smartcard technology, where process geometries generally trail the mainstream by about 3 or 4 generations…and why are there 20 bondable pads on what should be an 8-pad part?

Zooming in a bit on the die, we find some interesting details:

![](http://bunniestudios.com/blog/images/fakest_diemark2.jpg)

Well, this chip isn’t made by ST…it’s made by Fairchild Semiconductor (FSC). No bueno.

![](http://bunniestudios.com/blog/images/fakest_diemark1.jpg)

And in fact, the die within is a [Fairchild 74LCX244](http://www.fairchildsemi.com/pf/74/74LCX244.html) “Low Voltage Buffer/Line Driver with 5V Tolerant Inputs and Outputs”, a much cheaper piece of silicon than the reputed ST19CF68 that the package was marked to contain.

Perhaps the most interesting thing about these specimen is the quality of the package and the markings:

![](http://bunniestudios.com/blog/images/fakest_package.jpg)

Normally, remarked chips are pretty cheesy: they are sanded, painted over, or ground down before being marked, typically with just a silkscreen; rarely do you see a laser used to do the remarking.

These chips show no evidence of any kind of remarking per se. These are original markings — someone acquired blanks of the 74LCX244 chip, and programmed a production laser engraver to put a high-quality fake marking on an otherwise virgin package. I, too, would have been fooled by this up until the chip was decapsulated and examined under a microscope.

This leaves a lot of questions unanswered. How was someone able to acquire unmarked Fairchild silicon? Was it an insider, or was Fairchild sloppy and throwing away unmarked rejects without grinding them up or clipping off leads so they can’t be dumpster-dived and resold? The laser marking machine used isn’t one of the cheap desktop engravers either — the marks are done with a high-power raster engraver, and the engraving artwork is spot-on.

Then again, I shouldn’t be so surprised…I’ve seen brazen remarking of DIMMs in Saige market (Kingston seems to be a popular target for fakes), and many of the counterfeiters openly display their arsenal of professional-quality thermal transfer label printers and hologram stickers at their disposal.

If fakes of this quality become more common, this could present a problem for the supply chain. Clearly, whoever did this, can fake just about any chip they want, and they are gradually finding their way into the US market. Resellers, especially distributors that specialize in buying excess manufacturer inventory, implicitly trust the markings on a chip. I don’t think chip makers will go so far as to put anti-counterfeiting measures on chip markings, but this is definitely something that makes me wary.
