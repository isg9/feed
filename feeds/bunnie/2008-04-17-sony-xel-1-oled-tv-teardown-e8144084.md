---
title: Sony XEL-1 OLED TV Teardown
url: https://www.bunniestudios.com/blog/2008/sony-xel-1-oled-tv-teardown/
published: "2008-04-17T10:24:19Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=243
---

# Sony XEL-1 OLED TV Teardown

I was at the Embedded Systems Conference today in San Jose, and I saw a presentation where a [Sony XEL-1 OLED TV](http://www.sony.net/SonyInfo/News/Press/200710/07-1001E/index.html) was torn down live. It’s a sweet piece of technology, the crispest screen I’ve ever seen. Unlike LCDs, the screen doesn’t develop weird artifacts when you poke at it or twist it. I first saw it at CES in January, and then I got to lay my hands on it ever so briefly while in Japan a couple weeks ago…so tempted to get one…and then, I run into it today, again, by chance: the speaker’s luggage was lost so the teardown presentation, originally scheduled for yesterday, got moved to the day I was at the conference.

Unfortunately, I failed to note who the speaker and sponsoring organization was so I can’t properly credit them; if someone knows I’d appreciate a note in the comments. Kudos to the speaker; a teardown is a gutsy thing to do in front of a live audience, as hacking is not an easy performance art. I had to demonstrate a certain soldering procedure once at a factory in China. I had over a half dozen girls, each of them expert solder jockeys in their own right, literally breathing down my neck as they watched every move (and mistake) I made. I think the fact that they were probably better at soldering than I was made it an extra-stressful live demonstration.

![](http://www.sony.net/SonyInfo/News/Press/200710/07-1001E/01.jpg)

Fortunately, the speaker was kind enough to pass around the parts during the talk, so I managed to snag some nice hi-res photos of the insides of this device. I’ll share some of the better ones with you.

Most of the images are clickable to very high resolution versions.

[![](http://bunniestudios.com/blog/images/sony_xel1_oledback_sm.jpg)](http://bunniestudios.com/blog/images/sony_xel1_oledback.jpg)

Above is the back of the OLED panel. Across the bottom is a set of chip-on-flex TAB drivers; there is a Cyclone-II FPGA plus some DRAM memory. I count three switching regulators for bias generation, and a couple of beefy op-amps. The Fujitsu part is a 24 MHz 16-bit MCU with 384 kbytes of FLASH and 24 kbytes of DRAM. The local RTC is hooked up as well as the main oscillator. It also has a USB2.0 interface; the presenter seemed to think it was connected to the external USB port in the base unit somehow, although I can’t independently verify that. One thing that was noted during the presentation and by some of the show-goers is the excessive amount of thermal management material that was used in the display’s construction. Apparently, OLEDs suffer from “avalanche thermal failures”. I’m guessing from the name it means that once an OLED display gets hot beyond a certain point, it spirals into oblivion, burning itself out, so Sony took no chances and used all sorts of premium thermal compounds and heatsinks in the design of the device.

[![](http://bunniestudios.com/blog/images/sony_xel1_oledbacking_sm.jpg)](http://bunniestudios.com/blog/images/sony_xel1_oledbacking.jpg)

One interesting thing about the OLED display is that it’s actually transparent when in the off state. The amazing contrast ratio is due in part to this semi-soft, dark backing that’s bonded to the rear of the OLED display. You can see above where I have peeled the backing away to reveal what the actual panel looks like from behind. My guess is the dark backing is there to deal with light scattered away from the front, so that you can get that amazing 1,000,000 : 1 contrast ratio. It may also have some function in terms of distributing heat in the display. Since each OLED element is a tiny heater when it’s on, you can get very uneven thermal distributions on the glass depending upon the image pattern, so I could imagine that a thermally conductive backing is also important for reliability. However, the backing didn’t feel cold to touch like most highly thermally conductive compounds do, so maybe it is just an optical scattering material.

[![](http://bunniestudios.com/blog/images/sony_xel1_oledtransparent_sm.jpg)](http://bunniestudios.com/blog/images/sony_xel1_oledtransparent.jpg)

Above is a photo where I’ve held the panel in front of a desklamp, so you can see how transparent the OLED material is. It’s slightly less transparent than an undriven LCD panel.

[![](http://bunniestudios.com/blog/images/sony_xel1_mainboard_sm.jpg)](http://bunniestudios.com/blog/images/sony_xel1_mainboard.jpg)

Above is the mainboard inside the XEL-1. It’s chocked full of parts, and was covered with heatsinks. As you can see, there are multiple banks of DRAM on the board. The NEC D720101 is a USB host controller. The MB91305 in the upper left hand corner is 32-bit, 64 MHz embedded controller “most suitable for embedded applications, for example, DVD player, printer, TV…”. The Sony CXD9903GG appears to be an image formatter of some kind — perhaps performs image scaling, rate matching, filtering, etc. The [NEC D61162AF1](http://www.necel.com/digital_av/en/mpegdec_tv/d61162.html) is an impressive chip. It has four cores, an NEC VR5500 core running at 654 MIPS @ 327 MHz, a MIPS32 4KEc core running at 236 MIPS @ 196 MHz, and two audio CPUs, MIPS32 4KEm cores running at 236 MIPS @ 196 MHz. It also has a 2.6 GB/s DDR2 memory interface, a hardware MPEG stream transport engine, and a few other goodies alluded to, such as an AES/3DES processor, something involving Dolby, and a small 2-D graphics engine. Whatcha gonna do with all that junk, all that junk inside your trunk? I’ll tell you what I think they do with that in a bit.

In the top left, interestingly, is a 10W audio amplifier from TI (I can’t make out the part number in the photo exactly, so I’m just going by the speaker’s notes). It’s a 10W audio amplifier…that drives a pair of 1 W speakers. Again, that’s a lot of junk in the trunk…

[![](http://bunniestudios.com/blog/images/sony_xel1_mainboard_back_sm.jpg)](http://bunniestudios.com/blog/images/sony_xel1_mainboard_back.jpg)

Above is the backside of the mainboard. In the middle is a Realtek RTL8110SCL gigabit ethernet controller, and in the top right in a CXD9890 which I’m not quite sure what it does — there’s a forum on-line that speculates it is a video switch of some sort, which I might be willing to believe. There’s also a little Altera EPM3064A hiding out on the bottom … ahh, that brings me back in the day when I was an undergrad at the MIT Media Lab in the TV of the Future lab hand-packing logic into one of those things for the [Cheops](http://web.media.mit.edu/~wad/cheops_CSVT/cheops.html) project…haven’t seen one of those little CPLDs in forever.

[![](http://bunniestudios.com/blog/images/sony_xel1_teridian.jpg)](http://bunniestudios.com/blog/images/sony_xel1_teridian.jpg)

There’s one more circuit board in the XEL-1, which is a SmartCard reader board. There is [Teridian 73S8024RN](http://www.teridian.com/products/product_detail.cfm?product_key=62) smart card interface IC on it. I was explained, by a very patient clerk in Akihabara, that smart cards are required by digital TV subscribers in Japan.

So, I promised I’d get to answering what’s up with all that junk in the trunk of the XEL-1’s mainboard. The answer becomes obvious when you compare it against the insides of a [Sony Bravia KDL-20J3000](http://www.ecat.sony.co.jp/bravia/products/index.cfm?PD=27565) \[Japanese language link\]. The KDL-20J3000 is available only in Japan. I imported one a couple of weeks ago because I have developed a bizarre interest in Japanese TV hardware, even though I don’t watch any TV at all. The feature of the KDL-20J3000 that I found most interesting is that it has a built-in web browser; all of the high-end Japanese Bravia TVs have this feature and it’s not available in US Bravias. This is probably due in part to the fact that the web browser seems to be tied to a Japanese-language portal. A portal is necessary because there is no keyboard available on the remote; you must do all your browsing with a D-pad interface. I was curious about what they were using to make all that stuff go. Well, I wonder no longer. It’s that NEC D61162AF1; it certainly has the chops to run a little web browser, and then some!

[![](http://bunniestudios.com/blog/images/sony_bravia_mainboard_sm.jpg)](http://bunniestudios.com/blog/images/sony_bravia_mainboard.jpg)

Above is a photo of the Bravia TV mainboard. In the top middle is the NEC D61162AF1; to the left is the exact same Fujitsu 32-bit micro used in the XEL-1, and below that is the exact same Sony CXD9903GG also used in the XEL-1.

[![](http://bunniestudios.com/blog/images/sony_bravia_cpu_sm.jpg)](http://bunniestudios.com/blog/images/sony_bravia_cpu.jpg)

[![](http://bunniestudios.com/blog/images/sony_bravia_reformat_sm.jpg)](http://bunniestudios.com/blog/images/sony_bravia_reformat.jpg)

[![](http://bunniestudios.com/blog/images/sony_bravia_teridian.jpg)](http://bunniestudios.com/blog/images/sony_bravia_teridian.jpg)

Above are some close-ups of the interesting parts of the Bravia mainboard. The audio amplifier is a TPA3100D2, a 20W class-D amp, probably the same, if not a related part to the one used in the XEL-1…

[![](http://bunniestudios.com/blog/images/sony_bravia_impedance_sm.jpg)](http://bunniestudios.com/blog/images/sony_bravia_impedance.jpg)

Above is the impedance testing coupons on the edge of the Bravia mainboard, featuring both differential and single-ended test lines. The Bravia PCB is pretty big, and there are a lot of high-speed signals running around on it, so it’s natural that they’d require controlled impedance. The impedance testing coupons are a method for testing if the PCB has adequate electrical integrity before putting components on it. Dunno why I’m attracted to test coupons. I also compulsively check out the scribe line test structures on the edges of silicon dies as well.

So what’s the XEL-1 doing with all that junk in its trunk? My guess is that the XEL-1 basically has the exact same image engine architecture as used in the Bravia line of TVs, but packed into that tiny case. I haven’t read about the XEL-1 having a built-in web browser, so it’s possible they just borrowed the whole core from the Bravia line but disabled the web browser feature. Unfortunately, I haven’t been able to actually *use* a XEL-1; all the times I’ve seen it are either in showrooms or as now, disassembled, in pieces.

Presumably, borrowing the high-end Bravia digital TV engine was all done in the name of time to market — use a volume design that’s proven, and just repackage it into a tiny board, and turn off the features you didn’t need. When you’re Sony and you’re charging $2,000 for a product, you can afford to do stuff like that!
