---
title: Safecast Geiger Counter Reference Design
url: https://www.bunniestudios.com/blog/2012/safecast-geiger-counter-reference-design/
published: "2012-03-14T20:08:44Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=2218
---

# Safecast Geiger Counter Reference Design

This past weekend marked the anniversary of the Tohoku-Oki earthquake that devastated Japan. I had not felt my blood so cold since I watched the twin towers fall almost a decade earlier. I still vividly remember the twisting knots I felt in my stomach as I watched the footage of a tsunami wiping out huge swathes of Japanese countryside. In a matter of hours, entire cities were washed off the map, leaving an eerie post-apocalyptic landscape of a few survivors weeping amongst twisted wreckage. Then, in the ensuing days, Fukushima Daiichi melted down, leaving in its wake one of the worst on-going radiation contamination crisis since Chernobyl.

I have good friends in Japan, and I visit often. I wanted to do something to help, but I didn’t know what I could do. I was connected by [Joi Ito](http://twitter.com/#!/joi) to [Safecast](http://blog.safecast.org/), and I joined the effort to build an open sensor network that could aggregate trustable, source-neutral radiation monitoring data. Safecast itself has many talented and hard working volunteers who have done a remarkable job of achieving their goals by instrumenting Japan with radiation monitors and aggregating data through [cleverly designed and rapidly deployable](http://blog.safecast.org/2011/05/bgeigie-ninja-a-new-and-improved-bgeigie/photo-may-11-5-34-18-pm/) mobile monitoring capabilities.

I decided my tiny contribution to the effort would be to design a radiation monitor suitable for everyday civilian use. This is a preventative/preparedness measure, addressing the long-term issue of empowering a civilian population with few available options for power generation to self-monitor their environment. The problem with the current crop of radiation monitors is that they are basically laboratory instruments: accurate & reliable, but bulky, expensive, and difficult to use, requiring a degree in nuclear physics to understand exactly what the readings meant. Another problem with crises like these is that while radiation monitoring is important, it’s something that is typically neglected by the civilian population until it is too late.

Therefore, the challenge set out before me was to design a new Geiger counter that was not only more intuitive and easier to use than the current crop, but was also sufficiently stylish so that civilians would feel natural carrying it around on a daily basis. Furthermore, it had to provide extensive logging capabilities, as radiation monitors are typically not turned “on” until after the fact. It also had to operate effectively in catastrophic conditions, i.e. in scenarios where internet and power have been cut for days. Finally, the data collected by the instrument had to pass any scrutiny thrown its way, and the collected data had to be traceable to a given instrument so that if its calibration is incorrect, its data can be selectively excluded without poisoning the entire dataset. Radiation monitoring is a politically sensitive subject, and certain parties have interests to manipulate the data one way or the other to promote their views with the public. Ad-hoc data collection networks suffer from the possibility that their efforts can be discredited by institutions with big budgets who find that the readings represent an inconvenient truth.

**Radiation sensing primer**

Radiation measurements are subtle, partly because radiation comes in many flavors. Many Geiger counters can only efficiently detect the most energetic kind of radiation, gamma radiation. This includes the Geiger counters frequently favored by government and regulatory agencies. However, there are weaker forms of radiation (alpha and beta) which often go overlooked that can also pose a human health risk, particularly if they are ingested or inhaled. These weaker forms of radiation are also by-products of a nuclear meltdown, and because they come from different isotopes they have different patterns of distribution and absorption in the environment.

Because of the diversity of radiation sources and their varying biological impact, it is very hard to determine if an environment is safe in the face of an elevated Geiger counter reading. However, improved historical and spatial distribution records of background radiation measurements can help identify when there is a spike in radiation, which is a clearer cause for concern.

In the interest of creating a complete solution for public health needs, a core design requirement of the new Geiger counter is to incorporate a sensor that could detect all three forms of radiation. This type of sensor is a “pancake” style Geiger tube, which has a large mica window that enables sensitivity to all three kinds of radiation. The ultimate selection of the [LND7317 pancake tube](http://www.lndinc.com/products/17/) plus iRover HV radiation sensing core influenced every aspect of the industrial design (ID) and internal electronics.

**There and Back Again: a Hacker’s Tale**

I thought it would be interesting to share not only the final design, but also the intermediate designs that were scrapped en route to achieving a final design. Design is an iterative process, where one has to make difficult choices about what to include and more significantly what to leave out. It’s extremely rare to see what got left on the cutting room floor, but I saved my notes along the way so I could share them with you now.

**Initial Design Sketch**

[![](http://bunniestudios.com/safecast/safecast_concept1_sm.png)](http://bunniestudios.com/safecast/safecast_concept1.png)

Above is a rendering of the first design sketch, made back when Safecast had the name of “RDTN”. I do all my industrial design using Solidworks, a survival skill I picked up during my tenure at chumby designing consumer electronics. I came up with this in the first couple of weeks after the disaster. This design incorporated a [low-sensitivity tube from Sparkfun](http://www.sparkfun.com/products/8875), because at that time I did not understand the importance of using a pancake tube.

The biggest problem I wanted to solve with this design is user abandonment. Radiation leaks are thankfully rare events. However, this also means that when an event happens, there is typically a lack of pre-crisis background data against which to compare the post-crisis readings. Therefore, I wanted to build a device that people would be compelled to carry around every day and use for years at a time.

My thought is that the average consumer would have a hard time justifying carrying around yet another gadget in their pockets or purses for the sole purpose of measuring radiation. Within weeks or even days of getting nothing but “safe” readings, the Geiger counter would be forgotten and left at home to languish until after the next crisis.

One way to compel users to carry around a Geiger counter is to put it into something they already carry all the time. While it would be tough to convince a smartphone vendor to incorporate a very expensive and bulky Geiger tube, many smartphone users also carry around a spare battery pack, which they use almost every day. So, I thought it might be a good idea to trojan horse a Geiger tube into such a battery pack.

The sketch above demonstrates such an incarnation. This design is basically a battery pack that can charge a smartphone, but also incorporates a Geiger tube, an LED flashlight (handy in an emergency when there is no power), and some logging circuitry. The Geiger counter would upload its log data to the Safecast network whenever a user plugged in to charge the phone.

The design itself is minimalist, with a shape inspired by the steam cooling towers frequently used to iconify nuclear power in western media. The shape was chosen to remind us that sometimes we have no choice but to harvest the power of the atom, and a well-equipped and informed civilian data collection network is a key factor in trusting the safety of our power sources.

**A second iteration**

The first sketch had to be abandoned, primarily because the sensor it was designed around was too small to effectively measure alpha and beta radiation. After Safecast settled on the LND7317 Geiger tube as the standard reference sensor, I started re-designing the sensor around the new tube.

The problem with the larger, more sensitive sensor is that it was big – over a half-inch thick, and a couple inches in diameter. Below is a sketch of a design study aimed at creating the smallest possible Geiger counter that could also incorporate the large pancake tube. It’s about the size of a hockey puck, but a bit thicker. In order to keep the size and weight of the Geiger counter reasonable, I had to abandon any notion of a dual-purpose as a battery pack. Instead, I had to rely on “sex appeal” alone to compel users to carry the device around. I wanted to make the Geiger counter something unique and aesthetically pleasing, something you would enjoy carrying around frequently. I started from a minimalist design – the puck – and endeavored to design-out any outward indicators or displays. Hence in this sketch, the radiation measurement is provided by a set of super-high efficiency 7-segment LEDs that could shine the numbers through a seemingly opaque white shell. The design’s shape and feel was meant to be somewhere in between [“Eve”](http://disney.wikia.com/wiki/EVE) from WALL-E and an egg.

![](http://bunniestudios.com/safecast/safecast_puck3.png)

Unfortunately, this design, too, had to be abandoned because at the time when I was drawing up the sketches, I didn’t have detailed mechanical drawings of the LND7317 tube. When I was finally given a sample of the tube and drawings for it, I discovered there was not only the puck-like body, but also a nearly 1″ long protrusion for the cathode and anode. This completely destroyed any notion of building a puck-like sensor.

**Closing in on the final design**

Below is a rendering of an attempt to accommodate the accurate CAD model for the LND7317 into an ID that stayed faithful to the Eve/egg design inspirations. The puck was elongated to the minimum dimensions required to house all the internal components. Again, the hidden 7-segment LED display motif was employed.

[![](http://bunniestudios.com/safecast/safecast_7seg_sm.png)](http://bunniestudios.com/safecast/safecast_7seg.png)

**The final design**

After much discussion and review with the Safecast team, we decided that a key component of the user experience should be a graphic display, instead of a 7-segment LED readout. Therefore, a 128×128 pixel OLED panel was incorporated into the design. The OLED panel would be mounted behind a continuous outer shell, so there would be no seams or outward design features resulting from the introduction of the OLED. However, as the OLED is not bright enough to shine through an opaque white plastic exterior shell, a clear window had to be provided for the OLED. As a result, the naturally black color of the OLED caused the preferred color scheme of the exterior case to go from light colors to dark colors. User interaction would occur through a captouch button array hidden behind the same shell, with perhaps silkscreen outlines to provide hints as to where the buttons were underneath the shell. I had originally resisted the idea of using the OLED because it’s very expensive, but once I saw how much an LND7317 tube would cost in volume, I realized that it would be silly to not add a premium feature like an OLED. Due to the sensor alone, the retail price of the device would be in the hundreds of dollars; so adding an OLED display would help make the device “feel” a lot more valuable than a 7-segment LED display, even though the OLED’s presence is largely irrelevant to the core function of the apparatus.

[![](http://bunniestudios.com/safecast/safecast_final_render_sm.png)](http://bunniestudios.com/safecast/safecast_final_render.png)

The design also lacks any integrated radio connection. A popular request for the design was the incorporation of a bluetooth or zigbee style radio; however, a combination of a very stringent battery life goal (several months of standby time) and low manufacturing volumes meant that it was impractical to incorporate a radio into the device. It’s a slippery slope to start adding features like GPS and bluetooth – to add those features, you’d need to upgrade the microcontroller, at which point you’re basically building a very expensive, heavy and large cell phone with a geiger counter in it. Furthermore, the entire development effort was being done by an unpaid volunteer operating on a shoestring budget – Safecast isn’t Apple. So, rather than build a buggy cell phone that can sense radiation, I’d rather build an outstanding Geiger counter; hence the decision to focus efforts and resources on core functionality, with the sole allowance being the inclusion of the OLED + captouch array for improved UI. This is a controversial design decision and I fully expect to be chastised for it.

**The Prototypes**

Once the design was finished, the next step was to build prototypes. This is the really fun part, where you turn your ideas into something you can touch and hold.

[![](http://bunniestudios.com/safecast/safecast_proto_sm.jpg)](http://bunniestudios.com/safecast/safecast_proto.jpg)

[![](http://bunniestudios.com/safecast/safecast_proto_side_sm.jpg)](http://bunniestudios.com/safecast/safecast_proto_side.JPG)

The prototypes are made out of CNC-machined ABS (even the clear part!). The cosmetic moldings that go over the connectors were also built and they do fit, but because of their expense and fragility (CNC milled ABS lacks the robustness of injection-molded ABS), I try not to install them, even for glamor shots. To wit, the whole thing was done on a shoestring budget, as Safecast is a non-profit; two full prototypes were built, including PCB fab, assembly, and CNC milling for one and a half revisions of the cases, for a bit under $3k total.

An important point readers should note about this design is that I’m *not* manufacturing this Geiger counter reference design. My contribution is limited to design IP only. Practically speaking, I’d make a terrible Geiger counter supplier, because I don’t have the credibility or history in the industry. Instead, the design has been donated to the community, thereby enabling [International Medcom](http://medcom.com/), a business that has spent decades specialized in producing high-quality Geiger counters, to bring this to the market. If you’re interested in getting one of these, keep an eye on their website.

The final design features include:

LND7317 pancake tube + iRover HV board

STM32-based microcontroller; sufficient CPU power to digitally sign logs with a unique private key as a non-repudiation/anti-tamper measure

450 mAh Li-poly battery

3-axis accelerometer so sensor orientation can be recorded

128×128 color OLED display

6-button captouch array

“hold” button on the back to lock the captouch array and prevent false triggering of the power-hungry UI elements

lanyard attachment (important for the Japanese market)

microUSB port for charging and data upload interface, featuring an FTDI-based serial chipset capable of loading firmware into the microcontroller

3.5mm jack capable of bidirectional audio

embedded hall-effect sensor (to detect attachments, e.g. for occluding alpha or beta radiation)

audible event notification via piezo buzzer

low-power visual event notification via conventional LED

real-time clock

a high-quality entropy source ;-)

I am a proponent of open source hardware; so here’s the source files for my design! All of the following source files are licensed under [CC3.0-BY-SA](http://creativecommons.org/licenses/by-sa/3.0/) with my [XL1.0 automatic patent cross-license](http://bunniestudios.com/safecast/xl_crosslicense.pdf) rider (CC doesn’t address patents, so I invented my own rider that piggybacks on CC to ensure that any patents that may arise from this or its derivatives are automatically cross-licensed to the community).

[Altium design source](http://bunniestudios.com/safecast/elec/interface_electronics/altium_interface_electronics.zip) / [schematics](http://bunniestudios.com/safecast/elec/interface_electronics/safecast_ie3.pdf) / [gerbers](http://bunniestudios.com/safecast/elec/interface_electronics/gerbers_safecast_ie.zip) / [BOM](http://bunniestudios.com/safecast/elec/interface_electronics/safecast_ie3_bom.xls) for the mainboard electronics

[Altium design source](http://bunniestudios.com/safecast/elec/button_board/button_board.zip) / [schematics](http://bunniestudios.com/safecast/elec/button_board/safecast_buttonboard1.pdf) / [gerbers](http://bunniestudios.com/safecast/elec/button_board/gerbers_button_board.zip) / [BOM](http://bunniestudios.com/safecast/elec/button_board/safecast_bb1_bom.xls) for the buttonboard electronics

[Solidworks design source](http://bunniestudios.com/safecast/mech/safecast2.SLDPRT) / [IGES](http://bunniestudios.com/safecast/mech/safecast2.IGS) / [STEP](http://bunniestudios.com/safecast/mech/safecast2.STEP) for the industrial design

For those who don’t have 3D design tools, you can install [Solidwork’s free e-drawings viewer](http://www.solidworks.com/sw/products/free-cad-software-downloads.htm) and look at the [easm](http://bunniestudios.com/safecast/mech/test_assy3.easm) file, or if you run windows you can download [this executable](http://bunniestudios.com/safecast/mech/test_assy3.exe) and just run it

Of course, a hardware prototype is only the beginning – there’s a huge amount of effort remaining on the software. To bootstrap things, xobs and I have coded up a core demonstration system based on [Leaf Lab’s](http://leaflabs.com/) libmaple. You can [peruse the code](https://github.com/bunnie/libmaple) and/or download it at github. Basically, this demo system provides an architecture to easily register drivers and facilitate power management. The validation demo shown running on the prototype photos above indicate that all of the hardware features work. But, the software has yet to have a layer of polish and shine added in terms of the UI and power management optimization.

A key design goal electronics’ system design was to enable community participation. As such, I eschewed the use of JTAG adapters during development. Instead, hooks were provided in the hardware to enable the integrated FTDI USB-serial controller to flash the microcontroller’s firmware via a “bitbang” interface. As a result, anyone who has an interest in developing for this Geiger counter can simply plug it into their laptop’s USB port and start coding without any need for proprietary JTAG adapters or proprietary software to purchase, as the entire developer’s toolchain is available in source form. We were able to code up and test the entire functionality demo (including sleep/stop/standby power management) using nothing more than the USB-serial capability built into the design. As I write this, I realize I had neglected to upload the firmware loader to github, so [here’s a tarball](http://bunniestudios.com/safecast/fwload.tar.gz) for it; currently, the loader only runs under Linux and OSX.

I think there’s some fun things the community could do with the UI on a Geiger counter. At the very least, the microcontroller has sufficient power to play Tetris. Another whimsical thought was to build a subsystem that would play music out the audio port based upon the current radiation level — calm, ambient music in low-radiation environments escalating to death metal and the sound track of “Run Lola Run” at dangerous levels.

So that’s it! I hope that the design ultimately helps the people of Japan – or people anywhere in the world where radiation contamination may be a concern – to feel more empowered and in control of their situation.
