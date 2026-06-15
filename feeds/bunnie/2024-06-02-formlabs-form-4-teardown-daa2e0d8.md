---
title: Formlabs Form 4 Teardown
url: https://www.bunniestudios.com/blog/2024/formlabs-form-4-teardown/
published: "2024-06-02T19:05:24Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=7168
---

# Formlabs Form 4 Teardown

[Formlabs](https://formlabs.com/) has recently launched the fourth edition of their flagship SLA printer line, the [Form 4](https://formlabs.com/3d-printers/form-4/). Of course, I jumped on the chance to do a teardown of the printer; I’m grateful that I was able to do the same for the [Form 1](https://www.bunniestudios.com/blog/2013/formlabs-form-1-teardown/), [Form 2](https://www.bunniestudios.com/blog/2016/formlabs-form-2-teardown/), and [Form 3](https://www.bunniestudios.com/blog/2020/formlabs-form-3-teardown/) generations. In addition to learning a lot from the process of tearing down a single printer, I am also gaining a unique perspective on how a successful hardware startup matures into an established player in a cut-throat industry.

*Financial interest disclosure: Formlabs provides me two printers for the teardown, with no contingencies on the contents or views expressed in this post. I am also a shareholder of Formlabs.*

**A Bit of Background**

The past few years has seen a step-function in the competitiveness of products out of China, and SLA 3D printers have been no exception. The Form 4 sits at a pivotal moment for Formlabs, and has parallels to the larger geopolitical race for technological superiority. In general, Chinese products tend to start from a low price point with fewer features and less reliability, focusing on the value segment and iterating their way towards up-market opportunities; US products tend to start at a high price point, with an eye on building (or defending) a differentiated brand through quality, support, and features, and iterate their way down into value-oriented models. We now sit at a point where both the iterate-up and iterate-down approaches are directly competing for the same markets, setting the stage for the current trade war.

Being the first to gain momentum in a field also results in the dilemma of inertia. Deep financial and human capital investments into older technology often create a barrier to adopting newer technology. Mobile phone infrastructure is a poster child for the inertia of legacy: developing countries often have spectacular 5G service compared to the US, since they have no legacy infrastructure on the balance sheet to depreciate and can thus leapfrog their nations directly into mature, cost-reduced and modern mobile phone infrastructure.

In the case of 3D printers, Formlabs was founded in 2011, back when UV lasers were expensive, UV LEDs weren’t commercially viable, and full HD LCD screens (1080p) were just becoming mainstream. The most viable technology for directing light at the time was the galvanometer: a tiny mirror mounted on a motor that can adjust its angle with parts-per-thousands accuracy, tens of thousands of times a second. They invested in full custom galvanometer technology to create a consumer-priced SLA 3D printer – a technology that remained a gold standard for almost a decade, and powered three generations of Form printers.

![Form 1 and 2 light architecture](https://bunniefoo.com/bunnie/form4/form1_2.png)

Above is the light path architecture of the Form 1 and Form 2 printers. Both used a laser plus a pair of galvanometers to direct light in a 2-D plane to cure a single layer of resin through raster scanning.

![Form 3 light architecture](https://bunniefoo.com/bunnie/form4/form3.png)

The Form 3, introduced in 2019, pushed galvanometer technology to its limit. This used a laser and a single galvanometer to scan one axis, bounced off a curved mirror and into the resin tank, all in a self-contained module called a “light processing unit” (LPU). The second axis came from sliding the LPU along the rail, creating the possibility of parallel LPUs for higher throughput, and “unlimited” printing volume in the Y direction.

In the decade since Formlabs’ founding, LCD and LED technology have progressed to the point where full-frame LCD printing has become viable, with the first devices available in the range of 2015-2017. I got to play with my first LCD printer in 2019, around the time that the Form 3 was launched. The benefits of an LCD architecture were readily apparent, the biggest being build speed: an LCD printer can expose the entire build area in a single exposure, instead of having to trace a slice of the model with a single point-like laser beam. However, LCD technology still had a learning curve to climb, but manufacturers climbed it quickly, iterating rapidly and introducing new models much faster than I could keep up with.

Five years later and one pandemic in between, the Form 4 is being launched, with the light processing engine fully transitioned from galvanometer-and-laser to an LCD-and-LED platform. I’m not privy to the inside conversations at Formlabs about the change, but I imagine it wasn’t easy, because transitioning away from a decade of human capital investment into a technology platform can be a difficult HR challenge. The LPU was a truly innovative piece of technology, but apparently it couldn’t match the speed and cost of parallel light processing with an LCD. However, I do imagine that the LPU is still indispensable for high-intensity 3D printing technologies such as Selective Laser Sintering (SLS).

![Form 4 light architecture](https://bunniefoo.com/bunnie/form4/form4_arch.png)

Above is the architecture of the Form 4 – “look, ma, no moving parts!” – an entirely solid state design, capable of processing an entire layer of resin in a single go.

I’m definitely no expert in 3D printing technology – my primary exposure is through doing these teardowns – but from a first-principles perspective I can see many facial challenges around using LCDs as a light modulator for UV, such as reliability, uniformity, and build volume.

As their name implies, LCDs (liquid crystal displays) are built around cells filled with liquid crystal. Liquid crystals are organic molecules; [5CB](https://en.wikipedia.org/wiki/4-Cyano-4%27-pentylbiphenyl) is a textbook example of an LC compound.

![5CB molecule, from Wikipedia](https://bunniefoo.com/bunnie/form4/5cb.png)

Above is the structure of 5CB, snagged from its [Wikipedia page](https://en.wikipedia.org/wiki/4-Cyano-4%27-pentylbiphenyl); a few other LC molecules I looked up share a similar structure. I’m no organic chemist, but if you asked me “do you think a molecule like this might interact with intense ultraviolet light”, my answer is “definitely yes” – look at those aromatic rings!

A quick search seems to indicate that LCDs as printer elements can have a lifetime as short as a few hundred hours – which, given that the UV light is only on for a fraction of the time during a print cycle, probably translates to a few hundred prints. So, I imagine some conversations were had at Formlabs on how to either mitigate the lifetime issues or to make the machine serviceable so the LCD element can be swapped out.

Another challenge is the uniformity of the underlying LEDs. This challenge comes in two flavors. The first problem is that the LEDs themselves don’t project a uniform light cone – LEDs tend to have structural hot spots and artifacts from things such as bond wires that can create shadows; but diffusers and lenses incur optical losses which reduce the effective intensity of the light. The second is that the LEDs themselves have variance between devices and over time as they age, particularly high power devices that operate at elevated temperatures. This can be mitigated in part with smart digital controls, feedback loops, and cooling. This is helped by the fact that the light source is not on 100% of the time – in tests of the Form 4 it seems to be much less than 25% duty cycle, which gives ample time for heat dissipation.

Build volume is possibly a toss-up between galvo and LCD technologies. LCD resolutions can be made cheaply at extremely high resolutions today, so scaling up doesn’t necessarily mean coarser prints. However, you have to light up an even bigger area, and if your average build only uses a small fraction of the available volume, you’re wasting a lot of energy. On the other hand, with the Form 3’s LPU, energy efficiency scales with the volume of the part being printed: you only illuminate what you need to cure. And, the Form 3 can theoretically print an extremely wide build volume because the size in one dimension is limited only by the length of the rail for the LPU to sweep. One could conceivably 3D print the hood of a car with an LPU and a sufficiently wide resin tank! However, in practice, most 3D prints focus on smaller volumes – perhaps due to the circular reasoning of people simply don’t make 3D models for build volumes that aren’t available in volume production.

Despite the above challenges, Formlabs’ transition to LCD technology comes with the reward of greatly improved printing times. Prints that would have run overnight on the Form 3 now finish in just over an hour on the Form 4. Again, I’m not an expert in 3D printers so I don’t know how that speed compares against the state of the art today. Searching around a bit, there are some speed-focused printers that advertise some incredible peak build rates – but an hour and change to run the below test print is in net faster than my workflow can keep up with. At this speed, the sum of time I spend on 3D model prep plus print clean and finishing is more than the time it takes to run the print, so for a shop like mine where I’m both the engineer and the operator, it’s faster than I can keep up with.

[![sample print in clear resin](https://bunniefoo.com/bunnie/form4/sample-print-clear_sm.jpg)](https://bunniefoo.com/bunnie/form4/sample-print-clear.jpg)

**A Look around the Exterior of the Form 4**

Alright, let’s have a look at the printer!

![Unboxing the Form 4](https://bunniefoo.com/bunnie/form4/unbox.jpg)

The printer comes in a thoughtfully designed box, with ample padding and an opaque dust cover.

![Dust cover on the Form 4](https://bunniefoo.com/bunnie/form4/dustcover.jpg)

I personally really appreciate the detail of the dust cover – as I only have the bandwidth to design a couple products a year, the device can go for months without use. This is much easier on the eyes and likely more effective at blocking stray light than the black plastic trash bags I use to cover my printers.

![First impression](https://bunniefoo.com/bunnie/form4/first-impression.jpg)

The Form 4 builds on the design language of previous generation Form printers, with generous use of stamped aluminum, clean curves, and the iconic UV-blocking orange acrylic tank. I remember the first time I saw the orange tank and thought to myself, “that has to be really hard to manufacture…” I guess that process is now fully mature, as even the cheapest SLA printers incorporate that design touchstone.

A new feature that immediately leaps out to me is the camera. Apparently, this is for taking time lapses of builds. It increases the time to print (because the workpiece has to be brought up to a level for the camera to shoot, instead of just barely off the surface of the tank), but I suppose it could be useful for diagnostics or just appreciating the wonder of 3D printing. Unfortunately, I wasn’t able to get the feature to work with the beta software I had – it took the photos, but there’s something wrong with my Formlabs dashboard such that my Form 4 doesn’t show up there, and I can’t retrieve the time-lapse images.

Personally, I don’t allow any unattended, internet-connected cameras in my household – unused cameras are covered with tape, lids, or physically disabled if the former are not viable. I had to disclose the presence of the camera to my partner, which made her understandably uncomfortable. As a compromise, I re-positioned the printer so that it faces a wall, although I do have to wonder how many photos of me exist in my boxer shorts, checking in on a print late at night. At least the Form 4 is very up-front about the presence of the camera; one of the first things it does out of the box is ask you if you want to enable the camera, along with a preview of what’s in its field of view. It has an extremely wide angle lens, allowing it to capture most of the build volume in a single shot; but it also means it captures a surprisingly large portion of the room behind it as well.

![I see you seeing me](https://bunniefoo.com/bunnie/form4/camera-enable.jpg)

Kind of a neat feature, but I think I’ll operate the Form 4 with tape over the camera unless I need to use the time-lapse feature for diagnostics. I don’t trust ‘soft switches’ for anything as potentially intrusive as a camera in my private spaces.

![Backside of the Form 4](https://bunniefoo.com/bunnie/form4/back.jpg)

The backside of the Form 4 maintains a clean design, with another 3D printed access panel.

![Detail of service plug](https://bunniefoo.com/bunnie/form4/plug.jpg)

Above is a detail of the service plug on the back panel – you can see the tell-tale nubs on the inner surface of 3D printing. Formlabs is walking the walk by using their own printers to fabricate parts for their shipping products.

![Form 4’s butt](https://bunniefoo.com/bunnie/form4/butt.jpg)

The bottom of the printer reveals a fan and a massive heatsink underneath. This assembly keeps the light source cool during operation. The construction of the bottom side is notably lighter-duty than the Form 3. I’m guessing the all solid-state design of the Form 4 resulted in a physically lighter device with reduced mechanical stiffness requirements.

[![Imager surface](https://bunniefoo.com/bunnie/form4/imager-top_sm.jpg)](https://bunniefoo.com/bunnie/form4/imager-top.jpg)

Inside the printer’s cover, we get our first look at the pointy end of the stick – the imager surface. Unlike its predecessors, we’re no longer staring into a cavity filled with intricate mechanical parts. Instead we’re greeted with a textured LCD panel, along with some intriguing sensors and mounting features surrounding it. A side effect of no longer having to support an open-frame design is that the optical cavity within the printer is semi-sealed, with a HEPA-filtered centrifugal fan applying positive pressure to the cavity while cooling the optics. This should improve reliability in dusty environments, or urban centers where fine soot from internal combustion engines somehow finds their way onto every optical surface.

![A hidden toolkit](https://bunniefoo.com/bunnie/form4/allen-keys.jpg)

One minor detail I personally appreciate is the set of allen keys that come with the printer, hidden inside a nifty 3D printed holder. I’m a fan of products that ship with screwdrivers (or screwdriver-oids); it’s one of the feature points of the [Precursor](https://precursor.dev) hardware password manager that I currently market.

The Allen key holder is just one of many details that make the Form 4 far easier to repair than the Form 3. I recall the Form 3 being quite difficult to take apart; while it used gorgeous body panels with compound splines, the situation rapidly deteriorated from “just poking around” to “this thing is never going back together again”. The Form 4’s body panels are quite easy to remove, and more importantly, to re-install.

**A Deep Dive into the Light Path**

Pulling off the right body panel reveals the motherboard. Four screws and the panel is off – super nice experience for repair!

![Right panel removed](https://bunniefoo.com/bunnie/form4/mobo-context.jpg)

And below, a close-up of the main board (click for a larger version):

[![Mainboard view](https://bunniefoo.com/bunnie/form4/mobo-large_sm.jpg)](https://bunniefoo.com/bunnie/form4/mobo-large.jpg)

A few things come out at me on first glance.

First up, the Raspberry Pi 4 compute module. From a scrappy little “$35 computer” put out by a charity originally for the educational market, Raspberry Pi has taken over the world of single board computers, socket by socket. Thanks to the financial backing it had from government grants and donations as well as tax-free status as a charity, it was able to kickstart an unusually low-margin hardware business model into a profitable and sustainable (and soon to be publicly traded!) organization with economies of scale filling its sails. It also benefits from awesome software support due to the synergy of its charitable activities fostering a cozy relationship with the open source community. Being able to purchase modules like the Raspberry Pi CM with all the hard bits like high-speed DDR memory routing, emissions certification, and a Linux distro frees staff resources in other hardware companies (like Formlabs and my own) to focus on other aspects of products.

The next thing that attracted my attention is the full-on HDMI cable protruding from the mainboard. My first thought was to try and plug a regular monitor into that port and see what happens – turns out that works pretty much exactly as you’d expect:

[![Image of a layer on a 4K monitor](https://bunniefoo.com/bunnie/form4/lcd-image_sm.jpg)](https://bunniefoo.com/bunnie/form4/lcd-image.jpg)

In the image above, the white splotches correspond to the areas being cured for the print – in this case, the base material for the supports for several parts in progress.

![Layer as seen in Preform](https://bunniefoo.com/bunnie/form4/layer-2.png)

Above is a view of the layer being printed, as seen in the Preform software.

![Context of the print](https://bunniefoo.com/bunnie/form4/more-layers.jpg)

And above is a screenshot showing some context of the print itself.

The main trick is to boot the Form 4 with the original LCD plugged in (it needs to read the correct EDID from the panel to ensure it’s there, otherwise you get an error message), and then swap out the cable to an external monitor. I didn’t let the build run for too long for fear of damaging the printer, but for the couple layers I looked at there were some interesting flashes and blips. They might hint at some image processing tricks being played, but for the most part what’s being sent to the LCD is the slice of the current model to be cured.

I also took the LCD panel out of the printer and plugged it into my laptop and dumped its EDID. Here are the parameters that it reports:

![EDID](https://bunniefoo.com/bunnie/form4/edid.png)

I was a little surprised that it’s not a 4K display, but actually, we have to remember that each “color pixel” is actually three monochrome elements – so it probably has 2520 elements vertically, and 4032 elements horizontally. While resolutions can go higher than that, there are likely trade-offs on fill-factor (portion of a pixel that is available for light transmission versus reserved for switching circuitry) that are negatively impacted by trying to push resolution unnecessarily high.

![Partially removed LCD panel](https://bunniefoo.com/bunnie/form4/panel-partial-remove.jpg)

The LCD panel itself is about as easy to repair as the side body panels; just 8 accessible screws, and it’s off.

![HDMI cable retaining clip](https://bunniefoo.com/bunnie/form4/hdmi-retainer.jpg)

Another minor detail I really enjoyed on the LCD panel is the 3D-printed retaining clip for the HDMI cable. I think this was probably made out of nylon on one of Formlabs’ own [SLS printers](https://formlabs.com/asia/3d-printers/fuse-1/).

[![Backside of LCD assembly](https://bunniefoo.com/bunnie/form4/lcd-assy-backside_sm.jpg)](https://bunniefoo.com/bunnie/form4/lcd-assy-backside.jpg)

Turn the LCD panel assembly over, and we see a few interesting details. First, the entire assembly is built into a robust aluminum frame. The frame itself has a couple of heating elements bonded to it, in the form of PCBs with serpentine traces. This creates an interesting conflict in engineering requirements:

- The resin in the tank needs to be brought to temperature for printing
- The LCD needs to be kept cool for reliability
- Both need to be in intimate contact with the resin

Formlabs’ solution relies on the intimate contact with the resin to preferentially pull heat out of the heating elements, while avoiding overheating of the LCD panel, as shown below.

![Annotated view of the imager](https://bunniefoo.com/bunnie/form4/imager-annotated.jpg)

The key bit that’s not obvious from the photo above is that the resin tank’s lower surface is a conformal film that presses down onto the imaging assembly’s surface, allowing heat to go from the heater elements almost directly into the resin. During the resin heating phase of the print, a mixer turns the resin over continuously, ensuring that conduction is the dominant mode of heat transfer into the resin (as opposed to a still resin pool relying on natural convection and diffusion). The resin is effectively a liquid heat sink for the heater elements.

Of course, aluminum is an excellent conductor of heat, so to prevent heat from preferentially flowing into the LCD, gaps are milled into the aluminum panel that go along the edges of the panel, save the corners which still touch for good alignment. Although the gaps are filled with epoxy, the thermal conduction from the heating elements into the LCD panel is presumably much lower than that into the resin tank itself, thus allowing heating elements situated mere millimeters away from an LCD panel to heat the resin, without overheating the LCD.

One interesting and slightly puzzling aspect of the LCD is the textured film applied to the top of the LCD assembly. According to the Formlabs website, this is a “release texture” which prevents a vacuum between the film and the LCD panel, thus reducing peel forces and improving print times. The physics of print release from the tank film is not at all obvious to me, but perhaps during release phase, the angle between the film and the print in progress plays a big role in how fast the film can be peeled off. LCDs are extremely flat and I can see that without such a texture, air bubbles could be trapped between the film and the LCD; or if no air bubbles were there, there could be a significant amount of electrostatic attraction between the LCD and the film that can lead to inconsistent release patterns.

That being said, the texture itself creates a bunch of small lenses that should impact print quality. Presumably, this is compensated in the image processing pipeline by pre-distorting the image such that the final projected image is perfect. I tried to look for signs of such compensation in the print layers when I hooked the internal HDMI cable to an external monitor, but didn’t see it – but also, I was looking at layers that already had some crazy geometry to it, so it’s hard to say for sure.

The LCD driver board itself is about what you’d expect: an HDMI to flat panel converter chip, plus an EDID ROM.

[![LCD driver board](https://bunniefoo.com/bunnie/form4/lcd-driver_sm.jpg)](https://bunniefoo.com/bunnie/form4/lcd-driver.jpg)

As a side note, an LCD panel – a thing we think of typically as the finished product that we might buy from LG, Innolux, Sharp, CPT, etc. – is an assembly built from a liquid crystal cell (LC cell) and a backlight, along with various films and a handful of driver electronics. A lot of the smaller players who sell LCD panels are integrators who buy LC cells, backlights, and electronics from other specialty vendors. The cell itself consists of the glass sheets with the transparent ITO wires and TFT transistors, filled with LC material. This is the “hard part” to make, and only a few large factories have the scale to produce them at a competitive cost. The orange thing we’re looking at in the Form 4 is more precisely described as an LC cell plus some polarizing films and a specialized texture on top. Building a custom LC cell isn’t profitable unless you have millions of units per year volume, so Formlabs had to source the LC cells from a vendor specialized in this sort of thing.

Hold the panel up to a neutral light source (e.g., the sun), and we can see some interesting behaviors.

The video above was taken by plugging the Form 4’s LCD into my laptop and typing “THIS IS A TEST” into a word processor (so it appears as black text on a white background on my laptop screen). The text itself looks wider than on my computer screen because the Formlabs panel is probably using square pixels for each of the R, G, and B channels. For their application, there is no need for color filters; it’s just monochrome, on or off.

I suspect the polarizing films are UV-optimized. I’m not an expert in optics, but from the little I’ve played with it, polarizing films have a limited bandwidth – I encountered this while trying to polarize IR light for [IRIS](https://www.bunniestudios.com/blog/2024/iris-infra-red-in-situ-project-updates/). I found that common, off-the-shelf polarizing films seemed ineffective at polarizing IR light. I also suspect that the liquid crystal material within the panel itself is tailored for UV light – the contrast ratio is surprisingly low in visible light, but perhaps it’s much better in UV.

I’m also a bit puzzled as to why rotating the polarizer doesn’t cause light to be entirely blocked in one of the directions; instead, the contrast inverts, and at 45 degrees there’s no contrast. When I try this in front of a conventional IPS LCD panel, one direction is totally dark, the other is normal. After puzzling over it a bit, the best explanation I can come up with is that this is an [IPS panel](https://en.wikipedia.org/wiki/IPS_panel), but only one of the two polarizing films have been applied to the panel. Thus an “off” state would rotate the incoming light’s polarization, and an “on” state would still polarize the light, but a direction 90 degrees from the “off” state.

![IPS LCD, from Wikipedia](https://bunniefoo.com/bunnie/form4/Diagram_LCD_IPS.jpg)

Above is a diagram illustrating the function of an [IPS panel from Wikipedia](https://en.wikipedia.org/wiki/IPS_panel).

I could see maybe there is a benefit to removing the incoming light polarizer from the LCD, because this polarizer would have to absorb, by definition, 50% of the energy of the incident unpolarized light, converting that intense incoming light into heat that could degrade the panel.

However, I couldn’t find any evidence of a detached polarizer anywhere in the backlight path. Perhaps someone with a bit more experience in liquid crystal panels could illuminate this mystery for me in the comments below!

Speaking of the backlight path – let’s return to digging into the printer!

![Intermediate diffuser](https://bunniefoo.com/bunnie/form4/diffuser.jpg)

About an inch behind the LCD is a diffuser – a clear sheet of plastic with some sort of textured film on it. In the photo above, my hand is held at roughly the exit plane of the LED array, demonstrating the diffusive properties of the optical element. My crude tests couldn’t pick up any signs of polarization in the diffuser.

Beneath the diffuser is the light source. The light source itself is a sandwich consisting of a lens array with another laminated diffuser texture, a baffle, an aluminum-core PCB with LED emitters, and a heat sink. The heat sink forms a boundary between the inside and outside of the printer, with the outside surface bearing a single large fan.

Below is a view of the light source assembly as it comes out of the printer.

![Light source assembly](https://bunniefoo.com/bunnie/form4/light-source-assembly.jpg)

Below is some detail of the lens array. Note the secondary diffuser texture film applied to the flat surface of the film.

![Detail of the lens array](https://bunniefoo.com/bunnie/form4/lens-detail.jpg)

Below is a view of the baffle that is immediately below the lens array.

[![View of the baffle](https://bunniefoo.com/bunnie/form4/baffle_sm.jpg)](https://bunniefoo.com/bunnie/form4/baffle.jpg)

I was a bit baffled by the presence of the baffle – intuitively, it should reduce the amount of light getting to the resin tank – but after mating the baffle to the lens assembly, it becomes a little more clear what its function might be:

[![View of the baffle](https://bunniefoo.com/bunnie/form4/baffle-assy_sm.jpg)](https://bunniefoo.com/bunnie/form4/baffle-assy.jpg)

Here, we can see that the baffle is barely visible in between the lens elements. It seems that the baffle’s purpose might be to simply block sidelobe emissions from the underlying dome-lensed LED elements, thus improving light uniformity at the resin tank.

Beneath the baffle is the LED array, as shown below.

[![LED board overview](https://bunniefoo.com/bunnie/form4/led-board-overview_sm.jpg)](https://bunniefoo.com/bunnie/form4/led-board-overview.jpg)

And here’s a closer look at the drive electronics:

[![LED driver electronics](https://bunniefoo.com/bunnie/form4/led-board-drivers_sm.jpg)](https://bunniefoo.com/bunnie/form4/led-board-drivers.jpg)

There’s a few interesting aspects about the drive electronics, which I call out in the detail below.

[![LED driver electronics detail](https://bunniefoo.com/bunnie/form4/led-board-detail_sm.jpg)](https://bunniefoo.com/bunnie/form4/led-board-detail.jpg)

The board is actually two boards stacked on top of each other. The lower board is an aluminum-core PCB. If you look at the countersunk mounting hole, as highlighted by the buff-colored circle, you can see the shiny inner aluminum core reflecting light.

If you’re not already familiar with metal-core PCBs, my friends at [King Credie have a nice description](https://www.kingcredie.com/Ab_index_gci_58.html). From their site:

[![Cross section of a metal core PCB](https://bunniefoo.com/bunnie/form4/metal_core_xs.png)](https://www.kingcredie.com/Ab_index_gci_58.html)

The most economical (and most common) metal-core stack-up is a sheet of aluminum that has a single-layer PCB bonded to it. This doesn’t have as good thermal performance as a copper-core board with direct thermal heat pads, but for most applications it’s good enough (and much, much cheaper).

However, because the aluminum board is single-layer, routing is a challenge. Again, referring to the detail photo of the board above, the green circle calls out a big, fat 0-ohm jumper – you’ll see many of them in the photo, actually. Because of this topological limitation, it’s typical to see conventional PCBs soldered onto a metal-core PCB to instantiate more complicated bits of circuitry. The cyan circle calls out one of the areas where the conventional PCB is soldered down to the metal-core PCB using edge-plated castellations. This arrangement works, but can be a little bit tricky due to differences in the thermal coefficient of expansion between aluminum and FR-4, leading to long-term reliability issues after many thermal cycles. As one can see from this image, a thick blob of solder is used to connect the two boards. The malleability of solder helps to absorb CTE mismatch-induced thermal stresses.

The light source itself uses the [MAX25608B](https://www.analog.com/en/products/max25608.html), a chip capable of individually dimming up to 12 high-current LEDs in series (incidentally, I recently made a post covering the [theory behind this kind of lighting topology](https://www.bunniestudios.com/blog/2024/designing-the-light-source-for-iris/) for IRIS). This is not a cheap chip, given the Maxim brand and the AEC-Q100 automotive rating (although, the automotive rating means it can operate at up to 125 °C – a great feature for a chip mounted to a heat sink!), but I can think of a couple reasons why it might be worth the cost. One is that the individual dimming control could give Formlabs the ability to measure each LED in the factory and match brightness across the array, through a per-printer unique lookup table to dim the brightest outliers. Another is that Formlabs could simply turn off LEDs that are in “dark” regions of the exposure field, thus reducing wear and tear on the LCD panel. The PreForm software could track which regions of the LCD have been used the least, and automatically place prints in those zones to wear-level the LCD. Perhaps yet another reason is that the drivers are capable of detecting and reporting LED faults, which is helpful from a long-term customer support perspective.

To investigate the light uniformity a bit more, I defeated the tank-close sensors with permanent magnets, and inserted a sheet of white paper in between the resin tank and the LCD to capture the light exiting the printer just before it hits the resin.

However, a warning: don’t try this without eye protection, as the UV light put out by the printer can quickly damage your eyes. Fortunately, I happen to have a pair of these bad boys in my lab since I somewhat routinely play with lasers:

![Eye protection](https://bunniefoo.com/bunnie/form4/eye-protection.jpg)

Proper eye safety goggles will have their protection bandwidths printed on them: keep in mind that regular sunglasses may not offer sufficient protection, especially in non-visible wavelengths!

With the resin tank thus exposed, I was able to tell the printer to “print a cleaning sheet” (basically a single-layer, full-frame print) and capture images that are indicative of the uniformity of the backlighting:

![Uniformity test 1](https://bunniefoo.com/bunnie/form4/uniformity1.jpg)

Looks pretty good overall, but with a bit of exposure tweaking on the camera, we can see some subtle non-uniformities:

![Uniformity test 2](https://bunniefoo.com/bunnie/form4/uniformity2.jpg)

The image above has some bubbles in the tank from the mixer stirring the resin. I let the tank sit overnight and captured this the next day:

[![Uniformity test 3](https://bunniefoo.com/bunnie/form4/uniformity3_sm.jpg)](https://bunniefoo.com/bunnie/form4/uniformity3.jpg)

The uniformity of the LEDs changes slightly between the two runs, which is curious. I’m not sure what causes that. I note that the “cleaning pattern” doesn’t cause the fan to run, so possibly the LEDs are uncompensated in this special mode of operation.

The other thing I’d re-iterate is that without manually tweaking the exposure of the camera, the exposure looks pretty uniform: I cherry-picked a couple images so that we can see something more interesting than a solid bluish rectangle.

**Other Features of Note**

I spent a bit longer than I thought I would poking at the light path, so I’ll just briefly touch on a few other features I found noteworthy in the Form 4.

![Foam seal detail](https://bunniefoo.com/bunnie/form4/foam-seal.jpg)

I appreciated the new foam seal on the bottom of the case lid. This isn’t present on the Form 3. I’m not sure exactly why they introduced it, but I have noticed that there is less smell from the printer as it’s running. For a small urban office like mine, the odor of the resin is a nuisance, so this quality of life improvement is appreciated.

![HEPA filter](https://bunniefoo.com/bunnie/form4/hepa-filter.jpg)

I mentioned earlier in this post the replaceable HEPA filter cartridge on the intake of a blower that creates a positive pressure inside the optics path. Above is a photo of the filter. I was a little surprised at how loose-fitting the filter is; usually for a HEPA filter to be effective, you need a pretty tight fit, otherwise, particulates just go around the filter.

![Camera board](https://bunniefoo.com/bunnie/form4/camera.jpg)

The small plastic protrusion that houses the camera board (shown above) also contains the resin level sensor (shown below).

[![Resin level sensor](https://bunniefoo.com/bunnie/form4/resin-level-sensor_sm.jpg)](https://bunniefoo.com/bunnie/form4/resin-level-sensor.jpg)

The shape of the transducer on the sensor makes me think that it uses an ultrasonic time-of-flight mechanism to detect the level of the liquid. I’m impressed at the relative simplicity of the circuit – assuming I’m correct about my guess about the sensor, it seems that the STM32F303 microcontroller is directly driving the transducer, and the sole external analog circuit is presumably an LNA (low noise amplifier) for capturing the return echo.

The use of the STM32 also indicates that Formlabs probably hand-rolled the DSP pipeline for the ultrasound return signal processing. I would note that I did have a problem with the printer overfilling a tank with resin once during my evaluation. This could be due to inaccuracy in the sensor, but it could also be due to the fact that I keep the printer in a pretty warm location so the resin has a lower viscosity than usual, and thus it flows more quickly into the tank than their firmware expected. It could also be due to the effect of humidity and temperature on the speed of sound itself – poking around the [speed of sound](https://en.wikipedia.org/wiki/Speed_of_sound) page on Wikipedia indicates that humidity can affect sound speed by 0.1-0.6%, and 20C in temperature shifts things by 3% (I could find neither a humidity nor an air temperature sensor in the region of the ultrasonic device). This seems negligible, but the distance from the sensor to the tank is about 80mm and they are filling the tank to about 5mm depth +/- 1mm (?), so they need an absolute accuracy of around 2.5%. I suspect the electronics itself are more than capable of resolving the distance, as the time of flight from the transducer and back is on the order of 500 microseconds, but the environmental effects might be an uncompensated error factor.

Nevertheless, the problem was quickly resolved by simply pouring some of the excess resin back into the cartridge.

![Load cell underneath the resin cartridge](https://bunniefoo.com/bunnie/form4/load-cell.jpg)

Speaking of which, the Form 4 inherits the Form 3’s load-cell for measuring the weight of the resin cartridge, as well as the DC motor-driven pincer for squishing the dispenser head. The image above shows the blind-mating seat of the resin cartridge, with the load cell on the right, and the dispenser motor on the left.

[![RFID board](https://bunniefoo.com/bunnie/form4/rfid_sm.jpg)](https://bunniefoo.com/bunnie/form4/rfid.jpg)

A pair of 13.56 MHz RFID/NFC readers utilizing the TRF7970 allow the Form 4 to track consumables. One is used to read the resin cartridge, and the other is used to read the resin tank.

![Standby power of 26.7 watts](https://bunniefoo.com/bunnie/form4/standby-power.jpg)

Finally, for completeness, here’s some power numbers of the Form 4. On standby, it consumes around 27 watts – just a hair more than the Form 3. During printing, I saw the power spike as high as 250 watts, with a bit over 100 watts on average viewed at the plug; I think the UV lights alone consume over 100 watts when they are full-on!

**Epilogue**

Well, that’s a wrap for this teardown. I hope you enjoyed reading it as much as I enjoyed tearing through the machine. I’m always impressed by the thoroughness of the Formlabs engineering team. I learn a lot from every teardown, and it’s a pleasure to see the new twists they put on old motifs.

While I wouldn’t characterize myself as a hardcore 3D printing enthusiast, I am an occasional user for prototyping mechanical parts and designs. For me, the dramatically faster print time of the Form 4 and reduced resin odor go a long way towards reducing the barrier to running a 3D print. I look forward to using the Form 4 more as I improve and tune my [IRIS machine](https://www.bunniestudios.com/blog/2024/a-2-axis-multihead-light-positioner/)!
