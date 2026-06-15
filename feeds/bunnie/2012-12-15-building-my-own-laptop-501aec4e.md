---
title: Building my Own Laptop
url: https://www.bunniestudios.com/blog/2012/building-my-own-laptop/
published: "2012-12-15T16:46:32Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=2686
---

# Building my Own Laptop

We are building an open laptop, with some wacky features in it for hackers like me.

This is a lengthy project. Fortunately, ARM CPUs are getting fast enough, and [Moore’s Law is slowing down](http://www.bunniestudios.com/blog/?p=1863), so that even if it took a year or so to complete, I won’t be left with a woefully useless design. Today’s state of the art ARM CPUs — quad-core with GHz+ performance levels — is good enough for most day-to-day code development, email checking, browsing etc.

We started the design in June, and last week I got my first prototype motherboards, hot off the SMT line. It’s booting linux, and I’m currently grinding through the validation of all the sub-components. I thought I’d share the design progress with my readers.

Of course, a feature of a build-it-yourself laptop is that all the design documentation is open, so others of sufficient skill and resources can also build it. The hardware and its sub-components are picked so as to make this the most practically open hardware laptop I could create using state of the art technology. You can download, without NDA, the datasheets for all the components, and key peripheral options are available so it’s possible to build a complete firmware from source with no opaque blobs.

[![](http://bunniestudios.com/blog/images/novena_depop_clean_labels_sm.jpg)](http://bunniefoo.com/novena/novena_depop_clean_labels.jpg)

Above is an annotated diagram of the circuit board. The dimensions of the board are approximately 121mm x 150mm — sized to fit comfortably underneath a standard-sized laptop keyboard. The image above is rotated versus the installation orientation; the port farm is meant to be on the right hand side of the laptop, not on the bottom. The overall height of the board is just under 14mm, with the height being set by the thickness of an Ethernet connector. The thickness on my Lenovo T520 base portion is just under 24mm, so once we stack a keyboard and plastics on this it’ll be just about the same.

Here are some of the features of the laptop motherboard:

[Freescale iMX6 CPU](http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q&webpageId=129226228141673454B24A&nodeId=018rH3ZrDRB24A&fromPage=bunniestudios.com) — same footprint can support dual-lite and quad versions:

- Quad-core Cortex A9 CPU with NEON FPU @ 1.2 GHz
- Vivante GC2000 OpenGL ES2.0 GPU, 200Mtri/s, 1Gpix/s (\*)
- [NDA-free datasheet and programming manual](http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q&nodeId=018rH3ZrDRB24A&fpsp=1&tab=Documentation_Tab)

Internal memory:

- Boot from microSD firmware
- 64-bit, DDR3-1066 SO-DIMM, upgradable to 4GB
- SATA-II (3Gbps)

Internal ports & sensors:

- mini PCI-express slot (for blob-free [wifi](http://unex.com.tw/product/dnxa-125), [bluetooth](http://unex.com.tw/product/dhxa-195), [mobile data](http://www.roundsolutions.com/techdocs/ds/GTM661W.pdf), etc.)
- UIM slot for mPCIe mobile data cards
- Dual-channel LVDS LCD connector (up to QXGA (2048×1536) @ 60Hz resolution) with USB2.0 side-channel for a display-side camera
- Resistive touchscreen controller (note: captouch displays typically come with a controller)
- 1.1W, 8-ohm internal speaker connectors
- 2x USB2.0 internal connectors for keyboard and mouse/trackpad
- Digital microphone
- 3-axis accelerometer
- header for optional AW-NU137 wifi module (\*)

External ports:

- HDMI
- SD card reader
- headphone + mic port (compatible with most mobile phone headsets, supports sensing in-line cable buttons)
- 2x USB 2.0 ports, supporting high-current (1.5A) device charging
- 1Gbit ethernet

“Fun” features:

- 100 Mbit ethernet — dual Ethernet capability allows laptop to be used as an in-line packet filter or router
- USB OTG — enables laptop to spoof/fuzz ethernet, serial, etc. over USB via gadget interface to other USB hosts
- Utility serial EEPROM — for storing crash logs and other bits of handy data
- Spartan-6 CSG324-packaged FPGA — has several interfaces to the CPU, including a 2Gbit/s (peak) RAM-like bus — for your bitcoin mining needs. Or whatever else you might want to toss in an FPGA.
- 8x FPGA-driven 12-bit, 200ksps analog inputs
- 8x FPGA-driven digital I/O
- 8x FPGA-driven PWM headers, compatible with hobby ESC and PWM pinouts — enables direct interfacing with various RC motor/servo configurations & quad-copter controllers
- Raspberry-Pi compatible expansion header
- 13x CPU-driven supplemental digital I/Os
- 3x internal UART ports

Items marked with an asterisk (\*) require a closed-source firmware blob, but the system is functional and bootable without the blob.

In order to give maximum power management flexibility, the battery interface functions are implemented on a daughtercard. I co-opt a cheap and common SATA-style connector to route power and control signals between the mainboard and the daughtercard. To prevent users from accidentally plugging a hard drive into the battery port, I inverted the gender of the battery-SATA connector from the actual mass storage SATA-II connector. The current battery card is meant to work with the battery packs used by most RC enthusiasts — LiPo packs ranging from 2S1P to 4S1P (2-cell to 4-cell). RC packs are great because they are designed for super-fast charging. They are also cheap and easy to buy. For the board-side battery plug I decided to use the Molex connector found on classic disk drives, since they are cheap, common, and easy to assemble with simple tools. I couldn’t use a standard RC connector because the vast majority of them are designed for in-line use, and the few that have board mounts are too thick or too weird for use in this application.

[![](http://bunniestudios.com/blog/images/novena_batt_labels_sm.jpg)](http://bunniefoo.com/novena/novena_batt_labels.jpg)

The battery board can charge batteries at rates in excess of 4A. This means charging a 3-cell, 45Wh (4Ah) pack in about one hour. I’m estimating that a typical power consumption for a reasonable system configuration might be around 5-6W, so that’s 7-8 hours of runtime with a 1-hour charge time using that type of battery pack. Of course, since the whole laptop is user-configurable, typical power consumption is really hard to estimate — you could drop in a monster LCD and a power-hungry magnetic hard drive with loads of peripherals and the power consumption could be much higher. Of course, you can drop in a 100Wh battery pack if you wanted as well :)

Another cute feature of the battery board is that it can drive an analog panel meter. [Xobs](http://xoblo.gs) had suggested that it would be neat to embed a retro analog needle meter into the palmrest of the laptop to give a real-time display of power consumption. I thought it was a great idea, so I designed that in. Of course, the analog meter is driven by a DAC on the battery microcontroller, so it can be configured to perform a multitude of useful (or not so useful) analog read-outs, such as remaining runtime, battery voltage, temperature, the time (represented as an analog value), etc.

Next up is to spend a couple months validating all the features on the board — a long list of features to grind through indeed — and port drivers and a linux distro (no small task, but I’ll have [Xobs](http://xoblo.gs/)‘ skillful help). I also am looking forward to designing the enclosure. Probably for the first rev, I will do something out of laser-cut acrylic that is vaguely tablet-like, to avoid having to mess around with a friction clutch on version 1 of the plastics.

A detached [keyboard/trackpoint](http://support.lenovo.com/en_US/product-and-parts/detail.page?LegacyDocID=MIGR-73183) is attractive to me because I’ve always wanted a display I can “hang” on the seat in front of mine when sitting in an airplane or a bus — it’s a lot easier on the neck and the arrangement actually works *better* if the person in front reclines their seat.

Once I’ve got some experience integrating the whole thing, I’ll probably design a rev-2 case using CNC-cut ABS and aluminum. CNC cut ABS is almost as robust as injection molded ABS, and can produce reasonably intricate shapes. It’s also relatively economical to produce in single quantities. The CNC-cut design could be a clamshell design, or maybe some other funky design. Maybe I’ll try using wood and brass — who knows, the whole idea of making my own laptop is to play around with some new ideas!

It occurs to me that maybe other people might also be interested in owning a laptop like this, but don’t want to go through the trouble of fabricating their own circuit boards. If it seems like a few hundred folks are interested, I might be convinced to try a Kickstarter campaign in several months, once the design is stable and tested. However, I’m not looking to break any low-price records for this laptop — if you just want a cheap linux laptop you’re better off buying a netbook or EeePC. This is a low-volume, hand-crafted laptop made with uniquely open-source components, so the pricing would be consistent with such crafted goods.

For those interested in the source files for the current early prototype iteration of the design, bounce over to the [Novena wiki](http://www.kosagi.com/w/index.php?title=Novena_Main_Page), and keep an eye on [Xobs’ blog](http://xoblo.gs/). Novena (yet another Singaporean metro station, and also Latin for “nine”) is our stand-in codename for the laptop motherboard.
