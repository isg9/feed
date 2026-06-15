---
title: Introducing Precursor
url: https://www.bunniestudios.com/blog/2020/introducing-precursor/
published: "2020-09-19T14:55:30Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5921
---

# Introducing Precursor

> Precursor (pre·​cur·​sor \| \ pri-ˈkər-sər):
>
> 1\. one that precedes or gives rise to; a predecessor; harbinger
>
> 2\. a pocketable open development board

**Precursor** is a mobile, open source electronics platform. Similar to how a Raspberry Pi or an Arduino can be transformed into an IoT gadget with the addition of a couple breakout boards, some solder, and a bit of code, Precursor is a framework upon which you can assemble a wide variety of DIY mobile applications.

Precursor is unique in the open source electronics space in that it’s designed from the ground-up to be carried around in your pocket. It’s not just a naked circuit board with connectors hanging off at random locations: it comes fully integrated—with a rechargeable battery, a display, and a keyboard—in a sleek, 7.2 mm (quarter-inch) aluminum case.

![](https://bunniefoo.com/precursor/precursor_eightview.jpg)

### Precursor → Betrusted

Followers of my blog will recognize the case design from [Betrusted](https://betrusted.io), a secure-communication device. It’s certainly no accident that Precursor looks like Betrusted, as the latter is built upon the former. Betrusted is a great example of the kind of thing that you (and we) might want to make using Precursor. Betrusted is a huge software project, however, and it will require several years to get right.

[![](https://bunniefoo.com/precursor/precursor_to_betrusted.png)](https://betrusted.io)

Precursor, on the other hand, is ready today. And it has all of the features you might need to validate and test a software stack like the one that will drive Betrusted. We are also using the FPGA in Precursor to validate our SoC design, which will eventually give us the confidence we need to tape out a full-custom Betrusted ASIC, thereby lowering production costs while raising the bar on hardware security.

In the meantime, Precursor gives us a prototyping platform that we can use to work through user-experience challenges, and it gives **you** a way to implement projects that demand a secure, portable, trustable communications platform but that might not require the same level of hardware tamper resistance that a full-custom ASIC solution could provide.

And for developers, the best part is that Betrusted is 100% open source. As we make progress on the Betrusted software stack, we will roll those improvements back into Precursor, so you can count on a constant stream of updates and patches to the platform.

### Hackable. In a Good Way.

Precursor is also unique in that you can hack many aspects of the hardware without a soldering iron. Instead of a traditional ARM or AVR “System on Chip” (SoC), Precursor is powered by the software-defined hardware of a Field Programmable Gate Array (FPGA). FPGAs are a sea of basic logic units that users can wire up using a “bitstream”. Precursor comes pre-loaded with a bitstream that makes the FPGA behave like a RISC-V CPU, but you’re free to load up (or code up) any CPU you like, be it a 6502, an lm32, an AVR, an ARM, or something else. It’s entirely up to you.

This flexibility comes with its own set of trade-offs, of course. CPU speeds are limited to around 100 MHz, and complexity is limited to single-issue, in-order microarchitectures. It’s faster than any Palm Pilot or Nintendo DS, but it’s not looking to replace your smartphone.

### At Its Core

We describe bitstreams using a Python-based Fragmented Hardware Description Language (FHDL) called Migen, which powers the [LiteX](https://github.com/enjoy-digital/litex) framework. (Migen is to LiteX as GNU is to Linux, hence we refer to the combination as Migen/LiteX.) The framework is flexible enough that we can incorporate Google’s OpenTitan [SHA](https://github.com/betrusted-io/gateware/blob/master/gateware/sha2_litex.sv) and [AES](https://github.com/betrusted-io/gateware/blob/master/gateware/aes_opentitan.py) crypto-cores (written in SystemVerilog), yet powerful enough that we can natively describe a bespoke [Curve25519 crypto engine](https://ci.betrusted.io/betrusted-soc/doc/engine.html).

If you’ve ever wanted to customize your CPU’s instruction set, experiment with hardware accelerators, or make cycle-accurate simulations of retro-hardware, Precursor has you covered. And the best part is, thanks to Precursor’s highly integrated design philosophy, you can take all that hard work out of the lab and on the road.

### On the Inside

And if you’re itching for a excuse to break out your soldering iron or your 3D printer, Precursor is here to give you one. While its compact form factor might seem limiting at first, we’ve observed that 80% of projects involve adding just one or two domain-specific sensors or hardware modules to a base platform. And most of those additions come on breakout boards that require only a handful of signal wires.

![](https://bunniefoo.com/precursor/precursor_lte_module_example.jpg)

With eight GPIOs (configurable as three differential pairs and two single-ended lines) connected directly to the FPGA, Precursor’s battery compartment is designed to accommodate breakout boards. It also provides multiple power rails. You will find any number of third-party breakout boards with sensors ranging from barometers to cameras and radios ranging from BLE to LTE. Patch them in with a soldering iron, and you’re all set. The main trade-off is that, the more hardware you add, the less space you have left for your battery. Unless of course you build a bigger enclosure…

### On the Outside

If you need even more space or custom mounting hardware, the case is designed for easy fabrication using an aluminum CNC machine or a resin printer. Naturally, our case designs are open source, and the native Solidworks [CAD files](https://github.com/betrusted-io/betrusted-hardware/tree/master/case-v1) we provide are constructed such that the enclosure’s length and thickness are parameterized.

Furthermore, Precursor’s bezel is a [plain old FR-4 PCB](https://github.com/betrusted-io/betrusted-hardware/tree/master/bezel-v1), so if your application does not require a large display and a keyboard, you can simply remove them and replace the bezel with a full-sized circuit board. By way of example, removing the LCD and replacing it with a smaller OLED module would make room for a much larger battery while freeing up space for the custom hardware you might need to build, say, a portable, trustable, VPN-protected LTE hotspot.

### Come Have a Look!

If you’ve ever wanted to hack on mobile hardware, Precursor was made for you. By combining an FPGA dev board, a battery, a case, a display, and a keyboard into a single thin, pocket-ready package, it makes it easier than ever to go from a concept to a road-ready piece of hardware.

Precursor will soon be crowdfunding on Crowd Supply. Learn more about its specifications on [our pre-launch page](https://www.crowdsupply.com/sutajio-kosagi/precursor), and sign up for our mailing list so that you can take advantage of early-bird pricing when the campaign goes live.

### Meta

We’ve decided to do an extended pre-launch phase for the Precursor campaign to gauge interest. After all, we are in the middle of an unprecedented global pandemic, and one of the worst economic downturns in recorded history. It might seem a little crazy to try and fund the project now, but it’s also crazy to try and build trustable hardware that can hold up against state-level adversaries. We make decisions not on what is practical, but what is right.

The fact is the hardware is at a stage where we are comfortable producing more units and getting it into the hands of developers. The open question is if developers have the time, interest, and money to participate in our campaign. Initial outreach indicates there might be, but we’ll only find out for sure in the coming months. Precursor is not cheap to produce; I am prepared to accept a failed funding campaign as a possible outcome.

We’re also carefully considering alternate sources of funding, such as grants from organizations that share our values (such as [NLNet](https://nlnet.nl/PET/)) and commercial sponsors that will not attach conditions that compromise our integrity (you’ll notice the [Silicon Labs](https://www.crowdsupply.com/silicon-labs/iot-accelerator) banner at Crowd Supply). This will hopefully make the hardware more accessible, especially to qualified developers in need, but please keep in mind we are not a big corporation. As individual humans like you, we need to put food on the table and keep a roof over our heads. Our current plan is to offer a limited number of early bird units at a low price — so if you’re like me and worried about making ends meet next year, [subscribe to our mailing list](https://www.crowdsupply.com/sutajio-kosagi/precursor) so you can hopefully take advantage of the early bird pricing. And if you’re lucky enough to be in a stable situation, please consider backing the campaign at a higher pricing tier.

Over the coming months, I’ll be mirroring some of the more relevant posts from the campaign onto my blog, sometimes with additional commentary like this. There’s over two years of effort that have gone into building Precursor, and I look forward to sharing with you the insights and knowledge gained on my journey.
