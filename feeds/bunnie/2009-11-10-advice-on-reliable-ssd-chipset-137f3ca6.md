---
title: Advice on Reliable SSD Chipset?
url: https://www.bunniestudios.com/blog/2009/ssd-migration-advice-on-reliable-ssd-maker/
published: "2009-11-10T04:45:44Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=592
---

# Advice on Reliable SSD Chipset?

I spent the weekend transferring my data and applications to a new Crucial 256 GB M225 SSD, and after about 10 hours of operation, the hard drive simply failed. It failed while “hot” even — the hard drive lite jammed on, my MP3 stopped playing, and the system froze up — like my worst nightmare come true. How is this possible? It’s supposed to be solid state. It’s supposed to have greater than 1,000,000 hours MTBF. Yet, it’s true. No matter what laptop I put it into, I now simply get this on boot:

2100: HDD0 (Hard disk drive) initialization error (1).

The drive isn’t 100% dead per se. There is a little switch on the drive that puts it into configuration mode, and it will identify itself as a Yatapdong Barefoot device (instead of a Crucial device) in this mode. So presumably, the embedded controller is still alive and kicking, but even in this mode I can’t seem to reflash the drive’s firmware or re-initialize it to a state where the normal mode is functional. When I switch it back out of configuration mode, the BIOS refuses to enumerate the drive, and without enumeration I can’t even run a diagnostic on the device. It’s definitely not an OS issue — BIOS-level diagnostics simply refuse to recognize the drive.

Searching around in the [Crucial Forum](http://forums.crucial.com/t5/Solid-State-Drives-SSD/Crucial-M225-128GB-2-5-quot-SATA-II-Solid-State-Hard-Drive/m-p/3632), it seems these drives “have a high failure rate”. So much for a million-hour MTBF. I’m not sure exactly what’s causing it, because it is entirely solid state, and the SMT process used to build these drives are a very robust and well-proven technology. My guess is it’s got something to do with the firmware or the controller chip; they already have three firmware releases out for this drive, and disturbingly you can reflash these from inside the native OS — seems like a great candidate method for deeply embedding malware into a PC. Well, hopefully I can just return this drive for a full refund, since it failed so quickly.

At any rate, I’m wondering if anyone can give some advice on a good, reliable brand of SSD to use. The few hours I did spend with the SSD were quite positive; the performance boost is excellent, and a large number of my common work applications greatly benefited from the extremely fast access times. I’m still a bit spooked by the idea that these drives can fail so easily, but then again, fundamentally if I were in a jam I am equipped with the tools to recover the data — these devices use simple TSOP flash memory, so I suppose in the worst case I could dump the ROMs. Fortunately this one failed young, so the quickest solution is to just return for a refund and try something else.

Poking around a bit on-line, it seems that this Crucial drive uses the same Indilinx controller and firmware as the OCZ Vertex, the Patriot Torx, and the Corsair Extreme series (based on the “Yatapdong Barefoot” ID in configuration mode). On Amazon.com, I can see other users are experiencing [exactly the same issue](http://www.amazon.com/review/R26MD6SKX3RJRP/ref=cm_cr_rdp_perm) with a [Corsair Extreme Indilinx](http://www.amazon.com/Corsair-CMFSSD-256D1-Extreme-Indilinx-Internal/dp/B002LKCFTQ/ref=sr_1_2?ie=UTF8&s=electronics&qid=1257820978&sr=1-2) series device. So I think it’s safe to say I’d like to steer clear of a solution based on the [Indilinx chipset](http://www.indilinx.com/), at least until this issue is patched — ironically, the Indilinx website’s motto is “Beyond the Spin” (who are these guys, Fox News?), but it seems to me like their website’s got a lot more spin than substance. A link to a datasheet or firmware spec would be nicer than the marketing fluff.

Thanks in advance for the advice!
