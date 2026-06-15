---
title: Watermarks in Color Laserjet Printers
url: https://www.bunniestudios.com/blog/2005/watermarks-in-color-laserjet-printers/
published: "2005-08-20T19:11:50Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=48
---

# Watermarks in Color Laserjet Printers

While at [defcon XIII](http://www.defcon.org) (that was a blast, btw…too many fun things happened to write about!) I ran into [Seth Schoen](http://www.loyalty.org/~schoen/) from the [EFF](http://www.eff.org). He showed me one of the most frightining things I had seen in a while. Apparently, the US Secret Service has managed to convince a large number of color LaserJet manufacturers to [embed a watermark](http://www.eff.org/Privacy/printers/wp.php) into all printed pages. They are encoded as yellow dots that are virtually impossible to see without some kind of assistance, such as a microscope or spectrally pure light that is preferentially absorbed by the color yellow (such as the light emitted by a blue LED).

While there are legetimate uses for such a technology, there are a few problems with how the system was implemented. The most glaring problem is that the public wasn’t informed about it: how is it that the government managed to insert a tracking code on every color page I’ve printed for the past few years and I only found out now? Medical services, banks, ISPs, and others provide privacy policies to users (I’m not sure if they do it just because it’s good customer relations, or if they are legally required), yet I have never seen a printer owner’s manual document this potential privacy issue. This situation also leads to the paranoid speculation of what else has the government deployed that I don’t know about? Other problems, discussed in more detail at the [EFF’s website](http://www.eff.org/Privacy/printers/wp.php), include a lack of legislation to regulate the government’s ability to abuse this system.

If you’ve read this far, you can tell I’m not a fan of this technology. So, I’d like to figure out two things: exactly what does this code mean, and how to get around it. Neither problem is solved, but I’ll share bits and pieces of my progress to try and help anyone else who is also working on getting around this problem.

Toward figuring out what this code means, Seth informed me that currently they have an intern at the EFF with a microscope trying to write down the codes. Sounds like a painful task. To make this easier, I have taken a [Microtek S400 scanner](http://www.microtekusa.com/sms400.html) (the cheapest high resolution (4800 dpi) scanner I could find) and [modified it to emit blue LED light](http://www.bunniestudios.com/wordpress/?page_id=51) instead of a white light during the scanning process. [This hack](http://www.bunniestudios.com/wordpress/?page_id=51) enables you to get high-resolution full-page scans where the watermark is easily readable.

[![](http://bunniestudios.com/blog/images/2600n_sample.jpg)](http://bunniestudios.com/blog/images/2600n_sample_big.jpg)

Unretouched scanned image using the modified scanner; approx resolution is 1200 dpi (click to see full-size version). The watermarks are the small dots that seem randomly dispersed across the page, for example there is a dot to the lower left of and immediately on top of the “2” in “2600n”.
