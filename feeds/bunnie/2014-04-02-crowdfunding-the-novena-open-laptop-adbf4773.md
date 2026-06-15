---
title: Crowdfunding the Novena Open Laptop
url: https://www.bunniestudios.com/blog/2014/crowdfunding-the-novena-open-laptop/
published: "2014-04-02T15:58:14Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3657
---

# Crowdfunding the Novena Open Laptop

We’re launching a [crowdfunding campaign](https://www.crowdsupply.com/kosagi/novena-open-laptop) around our Novena open hardware computing platform. Originally, this started as a hobby project to build a computer just for me and [xobs](http://xobs.io/) – something that we would use every day, easy to extend and to mod, our very own Swiss Army knife. I’ve posted [here a couple of times](http://www.bunniestudios.com/blog/?tag=novena) about our experience building it, and it got a lot of interest. So by popular demand, we’ve prepared a crowdfunding offering and [you can finally be a backer](https://www.crowdsupply.com/kosagi/novena-open-laptop).

**Background**

[![](http://bunniefoo.com/novena/campaign/novenalaunch-frontside_sm.jpg)](http://bunniefoo.com/novena/campaign/novenalaunch-frontside.jpg)

Novena is a 1.2GHz, [Freescale quad-core ARM](http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q) architecture computer closely coupled with a [Xilinx FPGA](http://www.xilinx.com/products/silicon-devices/fpga/spartan-6/). It’s designed for users who want to modify and extend their hardware: all the documentation for the PCBs are [open and free to download](http://www.kosagi.com/w/index.php?title=Novena_PVT_Design_Source), and it comes with a variety of features that facilitate rapid prototyping.

We are offering four variations, and at the conclusion of the Crowd Supply campaign on May 18, all the prices listed below will go up by 10%:

- “Just the board” ($500): For crafty people who want to build their case and define their own style, we’ll deliver to you the main PCBA, stuffed with 4GiB of RAM, 4GiB microSD card, and an Ath9k-based PCIe wifi card. Boots to a Debian desktop over HDMI.
- “All-in-One Desktop” ($1195): Plug in your favorite keyboard and mouse, and you’re ready to go; perfect for labs and workbenches. You get the circuit board above, inside a hacker-friendly case with a Full HD (1920×1080) IPS LCD.

- “Laptop” ($1995): For hackers on the go, we’ll send you the same case and board as above, but with battery controller board, 240 GiB SSD, and a user-installed battery. As everyone has their own keyboard preference, no keyboard is included.
- “Heirloom Laptop” ($5000): A show stopper of beauty; a sure conversation piece. This will be the same board, battery, and SSD as above, but in a gorgeous, hand-crafted wood and aluminum case made by [Kurt Mottweiler](http://mottweilerstudio.com/) in Portland, Oregon. As it’s a clamshell design, it’s also the only offering that comes with a predetermined keyboard.

All configurations will come with Debian (GNU/Linux) pre-installed, but of course you can build and install whatever distro you prefer!

**Novena Gen-2 Case Design**

[![](http://bunniefoo.com/novena/campaign/novenalaunch-frontb.jpg)](http://bunniefoo.com/novena/campaign/novenalaunch-frontkey.jpg)

Followers of this blog may have seen a post featuring a prototype case design we put together last December. These were hand-built cases made from aluminum and leather and meant to validate the laptop use case. The design was rough and crafted by my clumsy hands – dubbed [“gloriously fuggly \[sic\]”](http://boingboing.net/2014/01/17/building-a-fully-open-transpa.html) – yet the public response was overwhelmingly positive. It gave us confidence to proceed with a 2nd generation case design that we are now unveiling today.

[![](http://bunniefoo.com/novena/campaign/novenalaunch-back_sm.jpg)](http://bunniefoo.com/novena/campaign/novenalaunch-back.jpg)

[![](http://bunniefoo.com/novena/campaign/novenalaunch-closed_sm.jpg)](http://bunniefoo.com/novena/campaign/novenalaunch-closed.jpg)

[![](http://bunniefoo.com/novena/campaign/novenalaunch-1_sm.jpg)](http://bunniefoo.com/novena/campaign/novenalaunch-1.jpg)

The first thing you’ll notice about the design is that the screen opens “the wrong way”. This feature allows the computer to be usable as a wall-hanging unit when the screen is closed. It also solves a major problem I had with the original clamshell prototype – it was a real pain to access the hardware for hacking, as it’s blocked by the keyboard mounting plate.

![](http://bunniefoo.com/novena/campaign/novena-open-rotate-small.gif)

Now, with the slide of a latch, the screen automatically pops open thanks to an internal gas spring. This isn’t just an open laptop — it’s a self-opening laptop! The internals are intentionally naked in this mode for easy access; it also makes it clear that this is *not a computer for casual home use*. Another side benefit of this design is there’s no fan noise – when the screen is up, the motherboard is exposed to open air and a passive heatsink is all you need to keep the CPU cool.

![](http://bunniefoo.com/novena/campaign/novena-bezel.jpg)

Another feature of this design is the LCD bezel is made out of a single, simple aluminum sheet. This allows users with access to a minimal machine shop to modify or craft their own bezels – no custom tooling required. Hopefully this makes adding knobs and connectors, or changing the LCD relatively easy. In order to encourage people to experiment, we will ship desktop and laptop devices with not one, but two LCD bezels, so you don’t have to worry about having an unusable machine if you mess up one of the bezels!

![](http://bunniefoo.com/novena/campaign/novena-enclosure-panel_project-body.jpg)

The panel covering the “port farm” on the right hand side of the case is designed to be replaceable. A single screw holds it in place, so if you design your own motherboard or if you want to upgrade in the future, you’re not locked into today’s port layout. We take advantage of this feature between the desktop and the laptop versions, as the DC power jack is in a different location for the two configurations.

![](http://bunniefoo.com/novena/campaign/novena_peek_array.jpg)

Finally, the inside of the case features a “Peek Array”. It’s an array of M2.5 mounting holes (yes, they are *metric*) populating the extra unused space inside the case, on the right hand side in the photo above. It’s named after [Nadya Peek](http://infosyncratic.nl/), a graduate student at MIT’s [Center for Bits and Atoms](http://fab.cba.mit.edu/). Nadya is a consummate maker, and is a driving force behind the CBA’s [Fab Lab](http://en.wikipedia.org/wiki/Fab_lab) initiative. When I designed this array of mounting bosses, I imagined someone like Nadya making their own circuit boards or whatever they want, and mounting it inside the case using the Peek Array.

[![](http://bunniefoo.com/novena/campaign/novena_speakerbox_sm.jpg)](http://bunniefoo.com/novena/campaign/novena_speakerbox.jpg)

The first thing I used the Peek Array for is the speaker box. I desire loud but good quality sound out of my laptop, so I 3D printed a speakerbox that uses 36mm mini-monitor drivers, and mounted it inside using the Peek Array. I would be totally stoked if a user with real audio design experience was to come up with and share a proper tuned-port design that I could install in my laptop. However, other users with weight, space or power concerns can just as easily design and install a more modest speaker.

![](http://bunniefoo.com/novena/campaign/novena_celia_chemmy.jpg)

I started the Gen-2 case design in early February, after [xobs](http://xobs.io) and I finally decided it was time to launch a crowdfunding campaign. With a bit of elbow grease and the help of a hard working team of engineers and project managers at my contract manufacturing partner, [AQS](http://aqs-inc.com) (that’s Celia and Chemmy pictured above, doing an initial PCBA fitting two weeks ago), I was able to bring a working prototype to San Jose and use it to give my keynote at [EELive today](http://www.eetimes.com/document.asp?doc_id=1320614).

**The Heirloom Design (Limited Quantities)**

One of the great things about open hardware is it’s easier to set up design collaborations – you can sling designs and prototypes around without need for NDAs or cumbersome legal agreements. As part of this crowdfunding campaign, I wanted to offer a really outstanding, no-holds barred laptop case – something you would be proud to have for years, and perhaps even pass on to your children as an heirloom. So, we enlisted the help of [Kurt Mottweiler](http://mottweilerstudio.com/) to build an “heirloom laptop”. Kurt is a designer-craftsman situated in Portland, Oregon and drawing on his background in luthiery, builds bespoke cameras of outstanding quality from materials such as wood and aluminum. We’re proud to have this offering as part of our campaign.

[![](http://bunniefoo.com/novena/campaign/heirloom-1_sm.jpg)](http://bunniefoo.com/novena/campaign/heirloom-1.jpg)

For the prototype case, Kurt is featuring rift-sawn white oak and bead-blasted-and-anodized 6061 aluminum. He developed a composite consisting of outer layers of paper backed wood veneer over a high-density cork core with intervening layers of 5.5 ounce fiberglass cloth, all bonded with a high modulus epoxy resin. This composite is then gracefully formed into semi-monocoque curves, giving a final wavy shape that is both light, stiff, and considers the need for air cooling.

[![](http://bunniefoo.com/novena/campaign/heirloom-2_sm.jpg)](http://bunniefoo.com/novena/campaign/heirloom-2.jpg)

The overall architecture of Kurt’s case mimics the industry-standard clamshell notebook design, but with a twist. The keyboard used within the case is wireless, and can be easily removed to reveal the hardware within. This laptop is an outstanding blend of tasteful design, craftsmanship, and open hardware. And, to wit, since these are truly hand-crafted units, no two units will be exactly alike – each unit will have its own grain and a character that reflects Kurt’s judgment for that particular piece of wood.

[![](http://bunniefoo.com/novena/campaign/heirloom-3_sm.jpg)](http://bunniefoo.com/novena/campaign/heirloom-3.jpg)

**How You can Help**

For the crowdfunding campaign to succeed, [xobs](http://xobs.io/) and I need a couple hundred open source enthusiasts to [back the desktop or standard laptop](https://crowdsupply.com/kosagi/novena-open-laptop) offering.

And that underlies the biggest challenge for this campaign – how do we offer something so custom and so complex at a price that is comparable to a consumer version, in low volumes? Our minimum funding goal of $250,000 is a tiny fraction of what’s typically required to recover the million-plus dollar investment behind the development and manufacture of a conventional laptop.

We meet this challenge with a combination of unique design, know-how, and strong relationships with our supply chain. The design is optimized to reduce the amount of expensive tooling required, while still preserving our primary goal of being easy to hack and modify. We’ve spent the last year and a half poring over three revisions of the PCBA, so we have high confidence that this complex design will be functional and producible. We’re not looking to recover that R&D cost in the campaign – that’s a sunk cost, as anyone is [free to download the source](http://www.kosagi.com/w/index.php?title=Novena_PVT_Design_Source) and benefit from our thoroughly vetted design today. We also optimized certain tricky components, such as the LCD and the internal display port adapter, for reliable sourcing at low volumes. Finally, I spent the last couple of months traveling the world, lining up a supply chain that we feel confident can deliver this design, even in low volume, at a price comparable to other premium laptop products.

To be clear, this is not a machine for the faint of heart. It’s an open source project, which means part of the joy – and frustration – of the device is that it is continuously improving. This will be perhaps the only laptop that ships with a screwdriver; you’ll be required to install the battery yourself, screw on the LCD bezel of your choice, and you’ll get the speakers as a kit, so you don’t have to use our speaker box design – if you have access to a 3D printer, you can make and fine tune your own speaker box.

If you’re as excited about having a hackable, open laptop as we are, please [back our crowdfunding campaign at Crowd Supply](https://crowdsupply.com/kosagi/novena-open-laptop), and follow [@novenakosagi](https://twitter.com/intent/user?screen_name=novenakosagi) for real-time updates.
