---
title: Winner of Name That Ware January 2008!
url: https://www.bunniestudios.com/blog/2008/winner-of-name-that-ware-january-2008/
published: "2008-03-09T11:35:41Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=229
---

# Winner of Name That Ware January 2008!

The ware for January 2008 was a Tangent Quattro internet radio alarm clock, and the winner is vt! Congratulations. Picking a winner, as usual, was not easy, but vt had both the most timely answer and an adequate set of follow-up explanations revealing the thought process behind the guess. Azer gets an honorable mention for using pixel scaling off of a known reference geometry (the USB plug) to determine the size of the device and narrow down the possibilities; I thought that was clever.

Some of the details of the Tangent Quattro were interesting. It’s basically a speaker that happens to have an embedded linux computer with wifi inside it — in other words, the speaker was designed first, and then the electronics were fitted around the acoustic chamber. I think that’s a good methodology for designing any integrated hi-fi device like this. On the back panel of the device is a slogan of sorts that I got a chuckle out of:

![](http://bunniestudios.com/blog/images/ntw_jan_08_tag.jpg)

Here is a photo of the Reciva module’s front side:

![](http://bunniestudios.com/blog/images/ntw_jan_08_module.jpg)

Since the processor is an ARM9 architecture device, I took the liberty of reading out the ROM and mounting its JFFS2 filesystem on a chumby, and poking around a bit. It’s interesting to see their method for storing configuration information such as WEP keys, access point settings, and alarms…but I digress.

![](http://bunniestudios.com/blog/images/ntw_jan_08_antenna.jpg)

Above is a photo of the antenna they use. You might recognize it — it’s a standard 802.11 access point antenna. It’s not a terrible idea to stick with things that just work, especially if you have the space and cost headroom to afford such an antenna.

![](http://bunniestudios.com/blog/images/ntw_jan_08_backpanel.jpg)

Above is a photo of the back panel CCA for the radio. The interesting part is how they implemented the 100baseT connection — it’s essentially the guts of a USB dongle laid out into the PCB (upper right hand corner of the CCA). This is an ever more popular approach, as I’m finding that USB has gotten so cheap and easy to integrate that it’s no longer just for external peripheral interconnect — it’s becoming usable as an inside-the-box interconnect standard as well.
