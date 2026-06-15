---
title: Use the Force (Feedback) to Solder Small Things
url: https://www.bunniestudios.com/blog/2025/use-the-force-feedback-to-solder-small-things/
published: "2025-09-13T14:48:01Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=7422
---

# Use the Force (Feedback) to Solder Small Things

Here’s a quick tip for people learning how to solder small components: don’t rely on your eyes. Instead, solder by *touch*: use the force-feedback available through your fingertips. Human fingers can reliably detect bumps on the order of microns in size – much finer than the resolution of sight, even with the assistance of a decent microscope.

I use my eyes to get the components “about there”, and then switch to using fingertips for feedback. Really focusing in and just relying on vision for alignment tends to increase hand-shake. At least for me, the neurological delay on the loop from eye-to-motor control is longer (and more noisy) than the loop on tactile-to-motor control. So, I keep my eyes open, but allow them to wander slightly so I’m able to pay more attention to what I’m feeling with my fingers.

Try this out: drag a soldering iron tip across PCB traces or soldermask while averting your eyes (closing your eyes entirely might be too disorienting without a bit of practice). As you drag across the traces, you’ll notice a “bump” every time you encounter one. Do the same for pins on tiny components. Practice with a cold tip to get a feel for things, then add heat; pay attention to how the textures change with temperature. Eventually, you’ll be able to tell when the tip of the iron has met its mark by how it feels. Sometimes you can tell you’ve made a good solder joint just by how the metal feels as it melts; you can also tell when your tip hits a pin connected to a copper plane because the solder on the tip will freeze briefly on contact with the cold copper.

Likewise, the reason I use fine-tipped titanium tweezers isn’t because it’s better for gripping components – it’s because the stiffness of the titanium gives intimate feedback about the texture of the PCB & components. It’s also the same reasoning I use bare surgical scalpels to cut traces; if you mount the blade in a handle, you lose tactile feedback on what the blade is doing.

The other pointer I have is that efficient soldering happens not at the geometric tip of the iron, but rather at the spot where the solder is liquid on the tip (and if you don’t have shiny, liquid solder on your tip, clean the tip and put a small dot there). I’ve observed a tendency for beginners to use soldering irons like ball point pens, but in reality the surface tension of solder always pulls the liquid away from the point of the tip. If your components and pads are oxide-free (e.g. through the use of flux), the “right” amount of solder from a much larger liquid solder reservoir will disperse onto the solder joint through the laws of physics (via surface tension), assuming that your board was made using a modern solder mask formulation that naturally repels liquid solder.

![](https://bunniefoo.com/bunnie/soldering-tips-sep25.jpg)

*Above, foreground: knife-edge soldering tip, with a dot of liquid solder near the point; background shows bare surgical scalpels and titanium tweezers*.

This is one of many reasons why I prefer to use the “knife edge” soldering tip versus conical tips; the knife-edge’s versatile shape has a natural landing spot for liquid solder adjacent to a thin linear edge, backed by ample heat capacity. The effective geometry of the soldering point can be varied by adjusting the angle of attack relative to the work piece: the very end of a knife-edge is effectively a conical tip, while the blade is effective at delivering lots of heat and solder. It is often the tip of choice at many factories that I’ve toured. In the hands of an experienced operator, one can balance throughput and precision without changing tips, and it’s good for everything from through hole to fine-pitch TQFP, while also being effective at desoldering a range of small components.
