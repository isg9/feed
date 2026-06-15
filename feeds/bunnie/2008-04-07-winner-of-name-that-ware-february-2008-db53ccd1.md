---
title: Winner of Name that Ware February 2008!
url: https://www.bunniestudios.com/blog/2008/winner-of-name-that-ware-february-2008/
published: "2008-04-07T12:25:26Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=232
---

# Winner of Name that Ware February 2008!

The solution to the Ware from last month is shown below. Below is the schematic overlaid on the transistors, and then the schematic redrawn in a more recognizable form, highlighting the cross-coupled inverter pair that forms the core of a typical static CMOS storage element.

![](http://bunniestudios.com/blog/images/ntw_feb_08_soln1.JPG)

![](http://bunniestudios.com/blog/images/ntw_feb_08_soln2.jpg)

In this circuit, P1/N4 form the output driver/buffer. P2/P3 form the storage pair.

A more detailed analysis including a breakout of the connectivity in the context photo can be found in [this PDF file](http://bunniestudios.com/blog/images/ntw_feb_08_soln.pdf).

The basic functionality of the device is probably a pulse stretcher, depending on how it’s connected to the rest of the circuit. The device is transparent while the enable is high, and in a memory state when D is low and enable is low. If the D is driven high while the enable is driven low, the device always goes to a 1 state, and stays in that state until the enable is high again while D is low, thus stretching out the pulse.

This device is part of an array that spans a spot between the random logic array and the RF section of a MIFARE RFID reader chip, specifically the MFRC530 by NXP. My best guess is that it’s part of an array that stores/processes baseband information coming back from the RF section of the IC. Shouts to Chris and Karsten for their help at looking at this device, and especially to [Flylogic](http://www.flylogic.net/blog/) for providing the awesome photos! If you like looking at chipshots and silicon security, you seriously need to check out the [Flylogic blog](http://www.flylogic.net/blog/).

The quality of the entries for last months’ contest are all very high. Unfortunately, I can’t quite declare a winner yet, because I don’t have Sii’s plaintext or sto’s plaintext. Sto’s schematics are correct, and it does call out the memory function of the device, but Sii looks like the answer might be correct, and the post is earlier than sto’s. pablot could also have a correct schematic, but since an answer wasn’t specifically named, it’s currently down to who has the correct answer between Sii and sto; if Sii’s answer is correct, Sii wins; otherwise, sto wins because of the correct schematics posted.

Thanks for playing! Post your plaintext in the comments, and the winner will be named in the comments.
