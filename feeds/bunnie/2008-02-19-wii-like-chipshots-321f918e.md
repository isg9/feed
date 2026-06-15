---
title: Wii Like Chipshots!
url: https://www.bunniestudios.com/blog/2008/wii-like-chipshots/
published: "2008-02-19T08:05:35Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=227
---

# Wii Like Chipshots!

I love looking inside chips, and [Flylogic](http://www.flylogic.net) takes some of the sweetest chip shots. bushing sent me some Wii chips to play with a few weeks ago, and Chris at Flylogic expertly decap’d and imaged them for me. I thought they were pretty neat, so here’s a couple of them to share with you:

[![](http://bunniestudios.com/blog/images/mx23l4005_20x_sm.jpg)](http://bunniestudios.com/blog/images/mx23l4005_20x.jpg)

The chip above is the Macronix mask ROM part inside the Wii. It also has some SRAM and a real time clock on-die. The large block on the left is the mask ROM, and the smaller block on the right is the SRAM. The top right has a fairly regular arrays of flip-flop like logic structures, so those are probably command or address registers for the chip.

[![](http://bunniestudios.com/blog/images/wii_eep_20x_sm.jpg)](http://bunniestudios.com/blog/images/wii_eep_20x.jpg)

The chip above is the serial EEPROM chip that’s flip-mounted onto the Hollywood package. The Hollywood GPU on the Wii actually consists of three silicon chips on a single substrate, as the image below shows. The serial EEPROM is indicated by the pink arrow.

![](http://bunniestudios.com/blog/images/hollywood_dies.jpg)

The bond pads still have the flip-mounting bumps on them, so they show up as large black circles in the photo. [Flylogic](http://www.flylogic.net) later removed the bumps using a neat hack with their wirebonder, and then rebonded the die into an 8-pin DIP so the contents could be read out with a conventional ROM burner. I found it particularly enlightening to see the ratio of logic versus the size of the actual memory array for the serial EEPROM (the memory array is the regular set of cells in the top-right corner). Essentially, at this capacity scale (2048 bits), you’re paying for a bunch of logic, and not much memory. Doubling the memory capacity would minimally impact the overall die size, since most of what’s on there looks to be flip flops for shift registers and command latches.
