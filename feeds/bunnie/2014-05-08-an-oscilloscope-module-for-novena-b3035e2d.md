---
title: An Oscilloscope Module for Novena
url: https://www.bunniestudios.com/blog/2014/an-oscilloscope-module-for-novena/
published: "2014-05-08T05:59:25Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3957
---

# An Oscilloscope Module for Novena

One of Novena’s most distinctive features is its FPGA co-processor. An FPGA, or Field Programmable Gate Array, is a sea of logic gates and memory elements that can be wired up according to hardware descriptions programmed in languages such as Verilog or VHDL. Verilog can be thought of as a very strictly typed C where every line of the code executes simultaneously. Thus, every bit of logic in Novena’s [Spartan 6 LX45 FPGA](http://www.xilinx.com/support/documentation/data_sheets/ds160.pdf) could theoretically perform a computation every clock cycle — all 43,000 logic cells, 54,000 flip flops, and 58 fixed-point multiply accumulate DSP blocks. This potential for massive parallelism underlies one half of the exciting prospects enabled by an FPGA.

The other exciting half of an FPGA relates to its expansive I/O capabilities. Every signal pin of an FPGA can be configured to comply with a huge range of physical layer specifications, from vanilla CMOS to high-speed differential standards such as TMDS (used in HDMI) and SSTL (used to talk to DDR memories). Each signal pin is also backed by a high speed SERDES (serializer/deserializer) and sophisticated clock management technologies. Need a dozen high-precision PWM channels for robotics? No problem, an FPGA can easily do that. Need an HDMI interface or two? Also no problem. Need a bespoke 1000 MT/s ADC interface? Simple matter of programming – and all with the *same set of signal pins*.

Novena also hangs a 2Gbit DDR3 memory chip directly off the FPGA. The FPGA contains a dedicated memory controller that talks DDR3 at a rate of 800MT/s over a 16-bit bus, yielding a theoretical peak memory bandwidth of 12.8 Gbits/s. This fast, deep memory is useful for caching and buffering data locally.

Thus, the FPGA can be thought of as the ultimate hardware hacking primitive. In order to unlock the full potential of the FPGA, we decided to bring most of the spare I/Os on the chip to a high speed expansion header. The high speed header is a bit less convenient than Arduino shield connectors if all you need to do is flash an LED, but as a trade-off the header is rated for signal speeds of over a gigabit per second per pin.

However, the GPBB (General Purpose Breakout Board) featured as one of the [Novena crowdfunding campaign stretch goals](https://www.crowdsupply.com/kosagi/novena-open-laptop/stretch-goals) resolves this inconvenience by converting the high speed signal format into a much lower performance but more convenient 0.1” pin header format, suitable for most robotics and home automation projects.

**Enter the Oscilloscope**

A problem that xobs and I frequently encounter is the need for a highly programmable, travel-friendly oscilloscope. There’s a number of USB scope solutions that don’t quite cut it in terms of analog performance and UX, and there are no self-contained solutions we know of today that allow us to craft stimulus-response loops of the type needed for fuzzing, glitching, [power analysis](http://ftp.cryptography.com/public/pdf/IntroToDPA.pdf), or other similar hardware hacking techniques.

Fortunately, Novena is an ideal platform for implementing a bespoke oscilloscope solution – which we’ve gone ahead and done. Here’s a video demonstrating the basic functionality of our oscilloscope solution running on Novena (720p version in [VP8](http://bunniefoo.com/novena/novescope/xobs-oscope-demo-720.webm) or [H.264](http://bunniefoo.com/novena/novescope/xobs-oscope-demo-720.mp4)):

Novena was plugged into the large-screen TV via HDMI to make filming the video a little bit easier.

In a nutshell, the oscilloscope offers two 8-bit channels at 1GSPS or one 8-bit channel at 2GSPS with an analog bandwidth of up to 900MHz. As a side bonus we also wired in a set of 10 digital channels that can be used as a simple logic analyzer. Here’s some high resolution photos of the oscilloscope expansion board:

[![](http://bunniefoo.com/novena/novescope/novescope-hs-adc_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-hs-adc.jpg)

[![](http://bunniefoo.com/novena/novescope/novescope-hs-adc-back_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-hs-adc-back.jpg)

Here’s [the schematics](http://bunniefoo.com/novena/novescope/novena-afe-hs-evt.PDF).

This combination of the oscilloscope expansion board plus Novena is a major step toward the realization of our dream of a programmable, travel-friendly oscilloscope. The design is still a couple revisions away from being production ready, but even in its current state it’s a useful hacking tool.

At this point, I’m going to geek out and talk about the tech behind the implementation of the oscilloscope board.

**Oscilloscope Architecture**

Below is a block diagram of the oscilloscope’s digital architecture.

[![](http://bunniefoo.com/novena/novescope/novescope-blockdiagram_sm.png)](http://bunniefoo.com/novena/novescope/novescope-blockdiagram.png)

The FPGA is configured to talk to an [ADC08D1020 dual 1GSPS ADC](http://www.digikey.com/product-search/en/integrated-circuits-ics/data-acquisition-analog-to-digital-converters-adc/2556291?k=adc08d1020), designed originally by National Semiconductor but now sold as TI. The interface to the ADC is a pair of 8-bit differential DDR busses, operating at up to 500MHz, which is demultiplexed 1:8 into a 64-bit internal datapath. Upon receipt of a trigger condition, the FPGA stores a real-time sample data from the ADC into local DDR3 memory, and later on the CPU can stream data out of the DDR3 memory via the Linux Generic Netlink API. Because the DDR3 memory’s peak bandwidth is only 1.6GSPS, deep buffer capture of 256 Msamples is only available for net sample rates below 1GSPS; higher sample rates are limited to the internal memory capacity of the FPGA, still a very usable 200 ksamples depth. [The design](https://github.com/bunnie/novena-afe-hs-fpga) is written in Verilog and consumes about 15% of the FPGA, leaving plenty of space for implementing other goodies like digital filters and other signal processing.

The ADC is clocked by an Analog Devices AD9520 PLL, which derives its time base from a TCXO. This PLL + TCXO combination gives us better jitter performance than the on-chip PLL of the FPGA, and also gives us more flexibility on picking sampling rates.

The power system uses a hybrid of boost, buck, and inverting switching regulators to bring voltages to the minimum-dropout required for point-of-use LDOs to provide clean power to sensitive analog subsystems. This hybrid approach makes the power system much more complex, but helps keep the power budget manageable.

Perhaps the most unique aspect of our oscilloscope design is the partitioning of the analog signal chain. Getting a signal from the point of measurement to the ADC is a major engineering challenge. Remarkably, the same passive probe I held in the 90’s is still a standard workhorse for scopes like my Tektronix TDS5104B almost a quarter century later. This design longevity is extremely rare in the world of electronics. With a bandwidth of several hundred MHz but an impedance measured in mega-ohms and a load capacitance measured in picofarads, it makes one wonder why we even bother with 50-ohm cables when we have stuff like oscilloscope probes. There’s [a lot of science behind this](http://archive.siliconchip.com.au/cms/A_111671/article.html), and as a result well-designed passive probes, such as the [Tektronix P6139B](http://www.amazon.com/gp/product/B005K0M6MK/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B005K0M6MK&linkCode=as2&tag=bunniestudios-20&linkId=DE2VNMPOTKB3Q32B)![](http://ir-na.amazon-adsystem.com/e/ir?t=bunniestudios-20&l=as2&o=1&a=B005K0M6MK), cost hundreds of dollars.

Unfortunately, high quality scope probes are made out of unicorn hair and unobtanium as far as I’m concerned, so when thinking about our design, I had to take a clean-sheet look at the problem. I decided to look at an active probe solution, whilst throwing away any notion of backward compatibility with existing scope probes.

I started the system design by first considering the wires (you can tell I’m a student of [Tom Knight](https://en.wikipedia.org/wiki/Tom_Knight_%28scientist%29) – one of his signature phrases is “it’s the wires, stupid!”). I concluded the cheapest high-bandwidth commodity cable that is also rated for a high insertion count is probably the SATA cable. It consists of two differential pairs and it has to support signal bandwidths measured in GHz, yet it [costs just a couple of bucks](http://www.amazon.com/gp/product/B001O4EPHA/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B001O4EPHA&linkCode=as2&tag=bunniestudios-20&linkId=JBSTZXSRORWC7ZQA)![](http://ir-na.amazon-adsystem.com/e/ir?t=bunniestudios-20&l=as2&o=1&a=B001O4EPHA). On the downside, any practical probing solution needs to present an impedance of almost a million times greater than that required by SATA, to avoid loading down the circuitry under test. This means we have to cram a high performance amplifier into a PCB that fits in the palm of your hand. Thankfully, Moore’s Law took care of that in the intervening decades from when passive oscilloscope probes were first [invented out of necessity](http://www.bunniestudios.com/blog/?p=46).

The [LMH6518](http://www.ti.com/lit/ds/symlink/lmh6518.pdf) is a single-chip solution for oscilloscope front-ends that is almost perfect for this scenario. It’s a 900 MHz, digitally controlled variable gain amplifier (VGA) with the added feature of an auxilliary output that’s well-suited for functioning as a trigger channel; conveniently, a SATA cable has two differential pairs, so we allocate one for measurement and one for trigger. We also strap a conventional 8-pin ribbon cable to the SATA cable for passing power and I2C.

The same LMH6518 VGA can be combined with a variety of front-end amplifiers to create a range of application-specific probes. We use a 1GHz FET op-amp (the ADA4817) to do the impedance transformation required of a “standard” digital oscilloscope. We use a relatively low impedance but “true differential” amplifier to measure voltages developed across a series sense resistor for power signature analysis. And we have a very high-impedance, high CMRR instrumentation amplifier front end for capturing signals developed across small loops and stubs of wire, useful for detecting parasitic electromagnetic emissions from circuits and cables.

[![](http://bunniefoo.com/novena/novescope/novescope-probe-dig_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-emc.jpg)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-dig-back_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-emc-back.jpg)

Above: [digital probe](http://bunniefoo.com/novena/novescope/novena-probe-dig-evt.PDF)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-dpa_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-dpa.jpg)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-dpa-back_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-dpa-back.jpg)

Above: [power signature analysis probe](http://bunniefoo.com/novena/novescope/novena-probe-dpa-evt.PDF)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-emc_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-emc.jpg)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-emc-back_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-emc-back.jpg)

Above: [sidechannel emissions probe](http://bunniefoo.com/novena/novescope/novena-probe-emc-evt.PDF)

However, the design isn’t quite perfect. The LMH6518 burns a lot of power – a little over a watt; and the pre-amp plus power regulators add about another watt overall to the probe’s power footprint. Two watts isn’t that bad on an absolute scale, but two watts in the palm of your hand is searing hot; the amplifier chip gets to almost 80C. So, I designed a set of custom aluminum heatsinks for the probes to help spread and dissipate the heat.

[![](http://bunniefoo.com/novena/novescope/novescope-probe-case_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-case.jpg)

[![](http://bunniefoo.com/novena/novescope/novescope-probe-case-back_sm.jpg)](http://bunniefoo.com/novena/novescope/novescope-probe-case-back.jpg)

When I handed the aluminum-cased probes to xobs, I warned him that the heat sinks are either going to solve the heat issue, or it’s going to turn the probes into a ball of flaming hot metal. Unfortunately, the heatsink gets to about 60C in still air, which is an ergonomic challenge – the threshold for pain is typically around 45-50C, so it’s very uncomfortable to hold the aluminum cases directly. It’s alright to hold the probes by the plastic connectors on the back, but this requires special training and users will instinctively want to hold a probe by its body. So, probably I’ll have to do some thermal optimization of the design and add either a heat pipe to a large heatsink off the probe body, or use a small fan to force air over the probes. It turns out just a tiny bit of airflow is all that’s need to keep the probes cool, but with passive convection alone they are simply too hot to handle. This won’t, of course, stop us from using them as-is; we’re okay with having to be a little bit careful to gain access to a very capable device. However, nanny-state laws and potentially litigious customers make it too risky to sell this solution to end consumers right now.

**Firmware Architecture**

xobs defined the API for the oscilloscope. The driver is based upon the [Generic Netlink API](http://en.wikipedia.org/wiki/Netlink) native to the Linux kernel, and relies upon the libnl-genl libraries for the user-space implementation. Out of the various APIs available in the Linux kernel to couple kernelspace to userspace, Netlink was the best match, as it is stream-oriented and inherently non-blocking. This API has been optimized for high throughput and low latency, since it is also the core of the IP network stacks that on servers push gigabits of bandwidth. It’s also more mature than the nascent Linux IIO subsystem.

In the case of [xobs’ driver](https://github.com/xobs/novena-scope-drivers), he creates a custom generic netlink protocol which he registers with the name “kosagi-fpga”. Generic netlink sockets support the concept of specific commands, and he currently supports the following:

`/* list of valid commands */ enum kosagi_fpga_commands { KOSAGI_CMD_UNSPEC, KOSAGI_CMD_SEND, KOSAGI_CMD_READ, KOSAGI_CMD_POWER_OFF, KOSAGI_CMD_POWER_ON, KOSAGI_CMD_FPGA_ASSERT_RESET, KOSAGI_CMD_FPGA_DEASSERT_RESET, KOSAGI_CMD_TRIGGER_SAMPLE, __KOSAGI_CMD_MAX, };`

The current implementation provisions two memory-mapped address spaces for the CPU to communicate with the FPGA, split along two different chip select lines. Chip Select 0 (CS0) is used for simple messages and register settings, while Chip Select 1 (CS1) is used for streaming data to and from the FPGA. Therefore, when the CPU wants to set capture buffer sizes, trigger conditions, or initiate a transfer, it communicates using CS0. When it wants to stream data from the FPGA, it will do so via CS1.

The core of the API is the KOSAGI\_CMD\_TRIGGER\_SAMPLE and KOSAGI\_CMD\_READ commands. To request a sample from the oscilloscope, the userspace program emits a KOSAGI\_CMD\_TRIGGER\_SAMPLE command to the kosagi-fpga Netlink interface. This will cause the CPU to communicate with the FPGA via the CS0 EIM memory space control registers, setting up the trigger condition and the transfer FIFO from the FPGA.

The userspace program will then emit a KOSAGI\_CMD\_READ command to retrieve the data. Upon receiving the read command, the kernel initiates a burst read from CS1 EIM memory space to a kernel buffer using memcpy(), which is forwarded back to the userspace that requested the data using the genlmsg\_unicast() Netlink API call. Userspace retrieves the data stream from the kernel by calling the nl\_recv() API call.

This call is currently configured to block until the data is available for the userspace program, but it can also be configured to timeout as well. However, a timeout is generally not necessary as the call will succeed in a fraction of a millisecond due to the high speed and determinism of the transfer interface.

In addition to handling data transfers, the kernel module implementing this API also handles housekeeping functions, such as configuring the FPGA and controlling power to the analog front end. FPGA configuration is handled automatically upon driver load (via insmod, modprobe, or udev) via the request\_firmware() API built into the Linux kernel. The FPGA bitstream is located in the kernel firmware directory, usually /lib/firmware/novena\_fpga.bit.

Power management functions have their own dedicated Netlink commands. Calling these commands causes the respective GPIO for the expansion connector power switch to be toggled. When the expanion connector is power-cycled, the module also resets the FPGA and reloads its firmware, allowing for a complete reset of the expansion subsystem without having to power cycle the CPU.

[![](http://bunniefoo.com/novena/novescope/usb_fullspeed_cropped_sm.png)](http://bunniefoo.com/novena/novescope/usb_fullspeed_cropped.png)

Above: a snippet of a trace captured by the scope when probing a full-speed USB data line.

xobs also wrote a wonderful [demo program in Qt for the oscilloscope](https://github.com/xobs/novena-scope-gui), and through this we were able to do some preliminary performance characterization. The rise-time performance of the probe is everything I had hoped for, and the very long capture buffer provided by the FPGA’s DDR3 memory enables a new dimension of deep signal analysis. This, backed with Novena’s horsepower, tight integration with Linux and a hackable architecture makes for a compelling – and portable – signal analysis solution for field work.

If the prospect of a a hackable oscilloscope excites you as much as it does us, please consider [backing our crowdfunding campaign for Novena](http://www.crowdsupply.com/kosagi/novena-open-laptop) and spreading the word to your friends; there’s only a few days left. Developing complex hardware and software systems isn’t cheap, and your support will enable us to focus on bringing more products like this to market.
