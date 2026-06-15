---
title: Name that Ware, July 2009
url: https://www.bunniestudios.com/blog/2009/name-that-ware-july-2009/
published: "2009-08-04T08:49:24Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=469
---

# Name that Ware, July 2009

The Ware for July 2009 is shown below. Click on the image for a much larger version.

[![](http://bunniestudios.com/blog/images/ntw_blankpcb1_sm.jpg)](http://bunniestudios.com/blog/images/ntw_blankpcb1.jpg)

I thought I’d try something new for this competition. Above is a scan of a completely blank PCB. I don’t actually know what it’s from. I just bought it from a vendor in the [Shenzhen mobile phone market](http://www.bunniestudios.com/blog/?p=283) who also didn’t know what it’s from — it was just used for shop decoration to attract customers. Since I actually don’t know what it’s for, I’ll have to rely on the depth of your comments to convince me. If nobody can convince me, then there would be no winner. On the other hand, it’s quite possible I could choose a winner who incorrectly identifies the ware, but that’s ok — this game has always been more about explanation and sharing thought processes, rather than about absolute accuracy.

I do think a blank PCB like this is fairly interesting to post for a number of reasons. First of all, it gives you a real appreciation for the sheer density of routing on a mobile phone motherboard. Second of all, this is a great example of an HDI-buildup, blind/buried via PCB. If you look at the large version of the scan, you can see the dimples where they used a laser, instead of a drill, to cut tiny vias into the outer layers of the circuit boards. There are almost no through-holes, and via-in-pad (typically forbidden when using through-hole vias) is very common in this design. Finally, this pair of PCBs is actually still in its manufacturing frame, which reveals a number of test structures that you never see (visible on the right hand side) that are used to verify that the PCB’s construction is correct.

You can also see in here how the two copies of the board are inverted across the horizontal mid-plane. I’m not 100% sure why they do this, but I suspect it has to do with balancing throughput on the SMT line. One side of the PCB is very dense with components and the other side isn’t. Since the board has components on both sides, you will have to flip the board over at least once to process each side. On high-throughput SMT lines, this means that you’ll have two (expensive) SMT [pick/place (or chip shooting) machines](http://www.bunniestudios.com/blog/?p=186) back to back, one to handle the top, and the other to handle the bottom after flipping. If you have both boards top-side up, then the SMT machine handling the denser top side would run much longer than the one handling the sparser bottom side, and you would effectively be idling one machine for a relatively long period of time. This would increase the amount of capital you have to spend to scale your production line and require greater amortization for a given throughput, ultimately driving up product cost (or reducing your net profit). Mirroring the board about a central axis also allows you to use the exact same programming for the two back-to-back SMT machines (separated by a robotic flipping mechanism) so you only have to optimize one SMT program, although at the expense of possibly having to incorporate a greater mix of parts within a single SMT machine.
