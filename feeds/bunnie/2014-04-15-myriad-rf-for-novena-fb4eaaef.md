---
title: Myriad RF for Novena
url: https://www.bunniestudios.com/blog/2014/myriad-rf-for-novena/
published: "2014-04-15T19:03:41Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3727
---

# Myriad RF for Novena

[![](http://bunniefoo.com/novena/Myriad-Novena_layout.png)](http://myriadrf.org/sdr-enabling-the-novena-open-hardware-laptop/)

This is so cool. Myriad-RF has created a port of their wideband software defined radio to Novena (read more [at their blog](http://myriadrf.org/sdr-enabling-the-novena-open-hardware-laptop/)). Currently, it’s just CAD files, but if there’s enough interest in SDR on Novena, they may do a production run.

The board above is based on the [Myriad-RF 1](http://myriadrf.org/myria-rf-board-1/). It is a fully configurable RF board that covers all commonly used communication frequencies, including LTE, CDMA, TD-CDMA, W-CDMA, WiMAX, 2G and many more. Their Novena variant plugs right into our existing high speed expansion slot — through pure coincidence both projects chose the same physical connector format, so they had to move a few traces and add a few components to make their reference design fully inter-operable with our Novena design. Their design (and the docs for the [transceiver IC](http://www.limemicro.com/products/LMS6002D.php)) is also fully [open source](https://github.com/myriadrf/myriadrf-boards), and in fact they’ve one-upped us because they use an open tool ( [KiCad](http://www.kicad-pcb.org/display/KICAD/KiCad+EDA+Software+Suite)) to design their boards.

I can’t tell you how excited I am to see this. One of our major goals in doing a [crowdfunding campaign](http://www.crowdsupply.com/kosagi/novena-open-laptop) around Novena is to raise community awareness of the platform and to grow the i.MX6 ecosystem. We can’t do everything we want to do with the platform by ourselves, and we need the help of other talented developers, like those at [Myriad-RF](http://myriadrf.org/), to unlock the full potential of Novena.
