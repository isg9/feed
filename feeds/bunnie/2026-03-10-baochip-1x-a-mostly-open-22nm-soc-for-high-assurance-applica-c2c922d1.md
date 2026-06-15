---
title: 'Baochip-1x: A Mostly-Open, 22nm SoC for High Assurance Applications'
url: https://www.bunniestudios.com/blog/2026/baochip-1x-a-mostly-open-22nm-soc-for-high-assurance-applications/
published: "2026-03-10T03:51:37Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=7513
---

# Baochip-1x: A Mostly-Open, 22nm SoC for High Assurance Applications

One of my latest projects is the Baochip-1x, a mostly-open, full-custom silicon chip fabricated in TSMC 22nm, targeted at high assurance applications. It’s a security chip, but far more open than any other security chip; it’s also a general purpose microcontroller that fills a gap in between the Raspberry Pi RP2350 (found on the Pi Pico2) and the NXP iMXRT1062 (found on the Teensy 4.1).

![](https://bunniefoo.com/baochip/dabao-thechip-narrow.jpg)

It’s the latest step in the [Betrusted](https://betrusted.io) initiative, spurred by [work I did with Ed Snowden](https://www.tjoe.org/pub/direct-radio-introspection/release/2) 8 years ago trying to answer the question of “can we trust hardware to not betray us?” in the context of mass surveillance by state-level adversaries. The Baochip-1x’s CPU core is descended directly from the FPGA SoC used inside [Precursor](https://precursor.dev), a device I made to keep secrets; designed explicitly to run [Xous](https://betrusted.io/xous-book/), a pure-Rust rethink of the embedded OS I helped write; and made deliberately compatible with [IRIS inspection](https://bunnie.org/iris/), a method I pioneered for non-destructively inspecting silicon for correct construction.

In a nutshell, the Baochip-1x is a SoC featuring a 350MHz Vexriscv CPU + MMU, combined with a I/O processor (“BIO”) featuring quad 700MHz PicoRV32s, 4MiB of nonvolatile memory (in the form of RRAM), and 2MiB of SRAM. Also packed into the chip are features typically found exclusively in secure elements, such as a TRNG, a variety of cryptographic accelerators, secure mesh, glitch sensors, ECC-protected RAM, hardware protected key slots and one-way counters.

![](https://bunniefoo.com/baochip/block-diagram.png)

The chip is fabricated using a fully-production qualified TSMC process using a dedicated mask set. In other words, this isn’t a limited-run MPW curiosity: Baochip’s supply chain is capable of pumping out millions of chips should such demand appear.

**Hardware Built to Run High-Assurance Software**

The Baochip-1x’s key differentiating feature is the inclusion of a Memory Management Unit (MMU). No other microcontroller in this performance/integration class has this feature, to the best of my knowledge. For those not versed in OS-nerd speak, the MMU is what sets the software that runs on your phone or desktop apart from the software that runs in your toaster oven. It facilitates secure, loadable apps by sticking every application in its own virtual memory space.

The MMU is a venerable piece of technology, dating back to the 1960’s. Its page-based memory protection scheme is well-understood and has passed the test of time; I’ve taught its principles to hundreds of undergraduates, and it continues to be a cornerstone of modern OSes.

![](https://bunniefoo.com/baochip/baochip1-atlas-vm.png)

*Above: Diagram illustrating an early virtual memory scheme from Kilburn, et al, “One-level storage system”, IRE Transactions, EC-11(2):223-235, **1962***

When it comes to evaluating security-oriented features, older is not always worse; in fact, withstanding the test of time is a positive signal. For example, the AES cipher is about 26 years old. This seems ancient for computer technology, yet many cryptographers recommend it over newer ciphers explicitly because AES has withstood the test of hundreds of cryptographers trying to break it, with representation from every nation state, over years and years.

I’m aware of newer memory protection technologies, such as [CHERI](https://en.wikipedia.org/wiki/Capability_Hardware_Enhanced_RISC_Instructions), PMPs, MPUs… and as a nerd, I love thinking about these sorts of things. In fact, [in my dissertation](https://bunniestudios.com/bunnie/phdthesis.pdf), I even advocated for the use of CHERI-style hardware capabilities and tagged pointers in new CPU architectures.

However, as a pragmatic system architect, I see no reason to eschew the MMU in favor of any of these. In fact, the MMU is composable with all of these primitives – it’s valid to have both a PMP and an MMU in the same RISC-V CPU. And, even if you’re using a CHERI-like technology for hardware-enforced bounds checking on pointers, it still doesn’t allow for transparent address space relocation. Without page-based virtual memory, each program would need to be linked to a distinct, non-overlapping region of physical address space at compile time, and you couldn’t have swap memory.

This begs the question: if the MMU is such an obvious addition, why isn’t it more prevalent? If it’s such an obvious choice, wouldn’t more players include it in their chips?

“Small” CPUs such as those found in embedded SoCs have lacked this feature since their inception. I trace this convention back to the introduction of the ARM7TDMI core in the 1990s. Back then, transistors were scarce, memory even more so, and so virtual memory was not a great product/market fit for devices with just a couple kilobytes of RAM, not even enough to hold a page table. The ARM7TDMI core’s efficiency and low cost made it a run-away success, shipping over a billion units and establishing ARM as the dominant player in the embedded SoC space.

Fast forward 30 years, and Moore’s Law has given us tens of thousands of times more capability; today, a fleck of silicon smaller than your pinky nail contains more transistors than a full-sized PC desktop from the 1990’s. Despite the progress, these small flecks of silicon continue to adhere to the pattern that was established in the 1990’s: small systems get flat memory spaces with no address isolation.

![](https://bunniefoo.com/baochip/baochip1-silicon-area.jpg)

*Above: Die shot of a modern 22nm system-on-chip (SoC). This fleck of silicon is about 4mm on a side and contains more transistors than a desktop PC from the 1990’s. Note how the logic region is more empty space than active gates by silicon area.*

The root cause turns out explicitly to be because MMUs are so valuable: without one, you can’t run Linux, BSD, or Mach. Thus, when ARM split their IP portfolio into the A, R, and M-series cores, the low-cost M-series cores were forbidden from having an MMU to prevent price erosion of their high-end A-series cores. Instead, a proprietary hack known as the “MPU” was introduced that gives some memory security, but without an easy path to benefits such as swap memory and address space relocation.

We’ve been locked into this convention for so long that we simply forgot to challenge the assumptions.

Thanks to the rise of open architecture specifications such as RISC-V, and fully-open implementations of the RISC-V spec such as the Vexriscv, I’m not bound by anyone’s rules for what can or can’t go onto an SoC. And so, I am liberated to make the choice to include an MMU in the Baochip-1x.

This naturally empowers enthusiasts to try and run Linux on the Baochip-1x, but we (largely Sean ‘xobs’ Cross and me) already wrote a pure-Rust OS called [“Xous”](https://betrusted.io/xous-book/) which incorporates an MMU but in a framework that is explicitly targeted towards small memory footprint devices like the Baochip-1x. The details of Xous are beyond the scope of this post, but if you’re interested, check out [the talk we gave at 39C3](https://media.ccc.de/v/39c3-xous-a-pure-rust-rethink-of-the-embedded-operating-system).

**“Now” Is Always the Right Time to Choose More Open Frameworks**

This couples into the core argument as to why a “mostly open RTL” SoC is the right thing for this moment in time. As a staunch advocate for open-source technologies, I would love to see a fully-open silicon stack, from the fabs-up. I’m heartened to see multiple initiatives working on fixing this problem, but it’s a hard problem. I estimate it could take more than a decade before we have a sufficiently robust open source silicon ecosystem to market economically competitive SoCs.

For those of us looking to create an embedded product today, that leaves only one practical option: continue to use Cortex-M ARM devices, and if we want hardware memory protection, we have to tune our software to their proprietary MPU. This means further entrenching our code bases in a proprietary standard. Do I really want to spend my time porting Xous to use ARM’s proprietary flavor of memory protection? Surely not.

Thus, I would argue that we simply can’t afford to wait for fully open source PDKs to come along. Given the opportunity to do a partially-open RTL tapeout today, versus waiting for the perfect, fully-open source solution, the benefit of taping out partially-open RTL SoCs today is crystal clear to me.

A partially-open SoC available today empowers a larger community that is interested in an open source future, even if they aren’t hardware experts. As a larger community, we can begin the process of de-leveraging ARM together, so that when economically viable, “truly open” silicon alternatives come to market, they can drop directly into a mature application stack. After all, software drives demand for silicon, not the other way around.

The good news is that on the Baochip-1x, everything that can “compute” on data is available for simulation and inspection, and it’s [already available on github](https://github.com/baochip/baochip-1x). The parts that are closed are components such as the AXI bus framework, USB PHY, and analog components such as the PLL, voltage regulators, and I/O pads.

Thus, while certain portions of the Baochip-1x SoC are closed-source, none of them are involved in the transformation of data. In other words, all the closed source components are effectively “wires”: the data that goes in on one side should match the data coming out the other side. While this is dissatisfying from the “absolute trust” perspective – one can’t definitively rule out the possibility of back doors in black-box wires – we can inspect its perimeter and confirm that, for a broad range of possibilities, it behaves correctly. It’s not perfect transparency, but it’s far better than the fully-NDA SoCs we currently use to handle our secrets, and more importantly, it allows us to start writing code for open architectures, paving a road towards an eventually fully-open silicon-to-software future.

**Hitchhiking for the Win**

Those with a bit of silicon savvy would note that it’s not cheap to produce such a chip, yet, I have not raised a dollar of venture capital. I’m also not independently wealthy. So how is this possible?

The short answer is I “hitchhiked” on a 22 nm chip designed primarily by [Crossbar, Inc](https://www.crossbar-inc.com/). I was able to include a CPU of my choice, along with a few other features, in some unused free space on the chip’s floorplan. By switching off which CPU is active, you can effectively get two chips for the price of one mask set.

![](https://bunniefoo.com/baochip/baochip1-floorlpan-sizes.jpg)

*Above: floorplan of the Baochip, illustrating the location and relative sizes of its 5 open-source CPU cores.*

For those who haven’t peeked under the hood of a System-on-Chip (SoC), the key fact to know is that the cost of modern SoCs is driven largely by peripherals and memory. The CPU itself is often just a small fraction of the area, just a couple percent in the case of the Baochip-1x. Furthermore, all peripherals are “memory mapped”: flashing an LED, for example, entails tickling some specific locations in memory.

Who does the tickling doesn’t matter – whether ARM or RISC-V CPU, or even a state machine – the peripherals respond just the same. Thus, one can effectively give the same “body” two different “personalities” by switching out their “brains” – by switching out their CPU cores, you can have the same physical piece of silicon run vastly different code bases.

The long answer starts a couple years ago, with Crossbar wanting to build a high-performance secure enclave that would differentiate itself in several ways, notably by fabricating in a relatively advanced (compared to other security chips) 22 nm process and by using their RRAM technology for non-volatile storage. RRAM is similar to FLASH memory in that it retains data without power but with faster write times and smaller (32-byte) page sizes, and it can scale below 40 nm – a limit below which FLASH has not been able to scale.

In addition to flexing their process superiority, they wanted to differentiate by being pragmatically open source about the design; security chips have been traditionally been wrapped behind NDAs, despite calls from users for transparency.

Paradoxically, open source security chips are harder to certify because the certification standards such as Common Criteria evaluates closed-source flaws as “more secure” than open-source flaws. My understanding is that the argument goes something along the lines of, “hacking chips is hard, so any barrier you can add to the up-front cost of exploiting the chip increases the effective security of the chip overall”. Basically, if the pen tester doing a security evaluation judges that a bug is easier to find and exploit if the source code is public, then, sharing the source code lowers your score. As a result, the certification scores of open source chips are likely worse than that of a closed source chip. And, since you can’t sell security chips to big customers without certifications, security chips end up being mostly closed source.

Kind of a crazy system, right? But if you consider that the people buying oodles and oodles of security chips are institutions like banks and governments, filled with non-technical managers whose primary focus is risk management, plus they are outsourcing the technical evaluation anyways – the status quo makes a little more sense. What’s a banker going to do with the source code of a chip, anyways?

Crossbar wanted to buck the trend and heed the call for open source transparency in security chips and approached me to help advise on strategy. I agreed to help them, but under one condition: that I would be allowed to add a CPU core of my own choice and sell a version of the chip under my own brand. Part of the reason was that Crossbar, for risk reduction reasons, wanted to go with a proprietary ARM CPU. Having designed chips in a prior life, I can appreciate the desire for risk reduction and going with a tape-out proven core.

However, as an open source strategy advisor, I argued that users who viewed open source as a positive feature would likely also expect, at a minimum, that the CPU would be open source. Thus I offered to add the battle-tested CPU core from the Precursor SoC – the Vexriscv – to the tapeout, and I promised I would implement the core in such a way that even if it didn’t work, we could just switch it off and there would be minimal impact on the chip’s power and area budget.

Out of this arrangement was born the Baochip-1x.

**Bringing the Baochip-1x Into the Market**

At the time of writing, wafers containing the Baochip-1x design have been fabricated, and hundreds of the chips have been handed out through an early sampling program. These engineering samples were all hand-screened by me.

However, that’s about to change. There’s currently a pod of wafers hustling through a fab in Hsinchu, and two of them are earmarked to become fully production-qualified Baochip-1x silicon. These will go through a fully automated screening flow. Assuming this process completes smoothly, I’ll have a few thousand Baochip-1x’s available to sell. More chips are planned for later in the year, but a combination of capital constraints, risk mitigation, and the sheer time it takes to go from blank silicon to fully assembled devices puts further inventory out until late in 2026.

[![](https://bunniefoo.com/baochip/dabao-sideways-front.jpg)](https://www.crowdsupply.com/baochip/dabao)

If you’re eager to play with the Baochip-1x and our Rust-based OS “Xous” which uses its MMU, consider checking out the “Dabao” evaluation board. A limited number are available for [pre-order today on Crowd Supply](https://www.crowdsupply.com/baochip/dabao). If this were a closed-source chip, this would be akin to a sampling or preview program for launch partners – one that typically comes with NDAs and gatekeepers controlling access. With Baochip, there’s none of that. Think of the campaign as a “friends and family” offering: an opportunity for developers and hackers to play with chips at the earliest part of the mass production cycle, before the supply chain is fully ramped up and sorted out.

この記事の日本語版はこちら → [日本語で読む](https://medium.com/ecosystembymakers/baochip-1x-138c430acf6b)
