---
title: Do Small-RAM Devices Have a Future?
url: https://blog.regehr.org/archives/541
published: "2011-06-12T15:21:38Z"
feed: regehr
guid: http://blog.regehr.org/?p=541
---

# Do Small-RAM Devices Have a Future?

Products built using microcontroller units (MCUs) often need to be small, cheap, and low-power. Since off-chip RAM eats dollars, power, and board space, most MCUs execute entirely out of on-chip RAM and flash, and in many cases don’t have an external memory bus at all. This piece is about small-RAM microcontrollers, by which I roughly mean parts that use only on-chip RAM and that cannot run a general-purpose operating system.

Although many small-RAM microcontrollers are based on antiquated architectures like Z80, 8051, PIC, and HCS12, the landscape is changing rapidly. More capable, compiler-friendly parts such as those based on ARM’s Cortex M3 now cost less than $1 and these are replacing old-style MCUs in some new designs. It is clear that this trend will continue: future MCUs will be faster, more tool-friendly, and have more storage for a given power and/or dollar budget. Today’s questions are:

> Where does this trend end up? Will we always be programming devices with KB of RAM or will they disappear in 15, 30, or 45 years?

I’m generally interested in the answer to these questions because I like to think about the future of computing. I’m also specifically interested because I’ve done a few research projects (e.g. [this](http://www.cs.utah.edu/~regehr/papers/p751-regehr.pdf) and [this](http://www.cs.utah.edu/~regehr/papers/pldi075-cooprider.pdf) and [this](http://www.cs.utah.edu/~regehr/papers/lctes062-yang.pdf)) where the goal is to make life easier for people writing code for small-RAM MCUs. I don’t want to continue doing this kind of work if these devices have no long-term future.

Yet another reason to be interested in the future of on-chip RAM size is that the amount of RAM on a chip is perhaps the most important factor in determining what sort of software will run. Some interesting inflection points in the RAM spectrum are:

- too small to target with a C compiler (< 16 bytes)
- too small to run multiple threads (< 128 bytes)
- too small to run a garbage collected language (< 128 KB)
- too small to run a stripped-down general-purpose OS such as Î¼Clinux (< 1 MB)
- too small to run a limited configuration of a full-fledged OS (< 32 MB)

These numbers are rough. It’s interesting that they span six orders of magnitude — a much wider range of RAM sizes than is seen in desktops, laptops, and servers.

So, what’s going to happen to small-RAM chips? There seem to be several possibilities.

**Scenario 1**: The incremental costs of adding transistors (in terms of fabrication, effect on packaging, power, etc.) eventually become so low that small-RAM devices disappear. In this future, even the tiniest 8-pin package contains an MCU with many MB of RAM and is capable of supporting a real OS and applications written in PHP or Java or whatever. This future seems to correspond to Vinge’s [*A Deepness in the Sky*](http://www.amazon.com/Deepness-Sky-Vernor-Vinge/dp/0812536355/), where the smallest computers, the [Qeng Ho localizers](https://blog.regehr.org/archives/255), are “scarcely more powerful than a Dawn Age computer.”

**Scenario 2:** Small-RAM devices continue to exist but they become so deeply embedded and special-purpose that they play a role similar to that played by 4-bit MCUs today. In other words — neglecting a very limited number of specialized developers — they disappear from sight. This scenario ends up feeling very similar to the first.

**Scenario 3:** Small-RAM devices continue to exist into the indefinite future; they just keep getting smaller, cheaper, and lower-power until genuine physical limits are reached. Eventually the small-RAM processor is implemented using nanotechnology and it supports applications such as machines that roam around our bloodstreams, or even inside our cells, fixing things that go wrong there. As an aside, I’ve picked up a few books on nanotechnology to help understand this scenario. None has been very satisfying, and certainly none has gone into the kind of detail I want to see about the computational elements of nanotechnology. So far the best resource I’ve found is Chapter 10 of Freitas’s [Nanomedicine Volume 1](http://www.amazon.com/Nanomedicine-Vol-I-Basic-Capabilities/dp/1570596808/).

This third scenario is, I think, the most interesting case, not only because small-RAM devices are lovable, but also because any distant future in which they exist is pretty interesting. They will be very small and very numerous — bear in mind that we already manufacture more MCUs per year than there are people on Earth. What sensors and actuators will these devices be connected to? What will their peripherals and processing units look like? How will they communicate with each other and with the global network? How will we orchestrate their activities?
