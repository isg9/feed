---
title: Lithography Projector Mod &#8211; Color Wheel Removal
url: https://sam.zeloof.xyz/lithography-projector-mod-color-wheel-removal/
published: "2018-04-05T02:24:18Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1639
---

# Lithography Projector Mod &#8211; Color Wheel Removal

When modifying a projector for photolithography, one thing to note is the UV transmission quality of the optics. Often, the stock color wheel in most DLP projects attenuates that UV significantly on the clear and blue sections. Thus, it is beneficial to remove it however the projector has RPM feedback (from motor back-EMF and/or photodiode) so simply removing it will result in the projector not starting up. A simple RC relaxation circuit can be built to emulate this signal (often 60 or 120hz) so that the projector will function properly.

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/02/D7V7630-e1522895557335-300x300.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/02/D7V7630-e1522895557335.jpg)

Color wheel removal

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/02/IMG_2641-e1522166624430-300x300.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/02/IMG_2641-e1522166624430.jpg)

Color wheel emulation

The circuit shown below is approximate. The 2.4uF capacitor and 3.3KΩ resistor set the frequency of oscillation.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3177-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3177.jpg)

Signal to emulate

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3178-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3178.jpg)

Components

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3179-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3179.jpg)

Circuit

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3180-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3180.jpg)

Yellow: output Purple: cap voltage

The completed circuit is probed and verified using an oscilloscope, using channels 1 and 2 which display the oscillator output and capacitor charging waveforms, respectively, in the last photo.

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/02/D7V7889-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/02/D7V7889.jpg)

Completed

In the end, the projector is “fooled” into operating with monochrome light output without a color wheel.
