---
title: Winner of Name that Ware October 2005!
url: https://www.bunniestudios.com/blog/2005/winner-of-name-that-ware-october-2005/
published: "2005-11-25T12:08:41Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=67
---

# Winner of Name that Ware October 2005!

Finally, a stumper! A lot of good speculation, but none of the answers were correct. Of all the answers, I’d say I’d have to pick Eric as the winner: these two phrases together were closest to the solution: “It looks more like the comparators were used to sense levels of things, and to trip logic” and “This board just reeks of some industrial control application, although I’ve seen similar boards in scientific equipment as well.”

This board is in fact a “Line and Spot” board from a scanning electron microscope (SEM). It is responsible for rendering an overlay of a crosshair or spot onto the video output of the SEM. Its input is a set of analog voltages that represent the horizontal and vertical position of the raster, and the resistor ladder + comparators (identified by many as a FLASH A/D converter, which it is in a rather odd sort of way) are responsible for toggling the overlay output on or off. The potentiometers are used to calibrate the dimensions of the crosshair.

I really like this board because it reminds us how a task that we take for granted today–rending a crosshair on a video screen–had to be implemented in a time before frame buffers, counters, and state machines were cheap and easy to implement, as they are today in an FPGA or an ASIC. Fortunately, this model of SEM came with full documentation, so you can see the schematics of this ware below:

[![](http://bunniestudios.com/blog/images/contest_10_2005_schematics_sm.jpg)](http://bunniestudios.com/blog/images/contest_10_2005_schematics.jpg)

Here are some photos of the SEM, backside:

![](http://bunniestudios.com/blog/images/sem_backside.jpg)

and the front side (it’s in my kitchen, that’s a keg fridge to the left):

![](http://bunniestudios.com/blog/images/sem_total.jpg)

Some may ask “why?” I respond with, “why not?”. Why do people buy old Mustangs and fix them up, when they can buy a more efficient, faster, and quieter car? Plus, once I get this up and running (it needs a lot of infrastructure to run properly) it’ll be a great tool in the old toolbox.
