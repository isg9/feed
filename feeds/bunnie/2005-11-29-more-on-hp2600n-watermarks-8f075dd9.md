---
title: More on HP2600N Watermarks
url: https://www.bunniestudios.com/blog/2005/more-on-hp2600n-watermarks/
published: "2005-11-29T07:25:01Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=72
---

# More on HP2600N Watermarks

So, earlier this month, I [posted a note](http://www.bunniestudios.com/wordpress/?p=66) that the serial EEPROM on the main board contained the serial number and formatter ID, among other things, for the HP2600N printer. A little tinkering around has revealed that the watermark is likely *not* in this EEPROM either! After poking through a few promising areas, I tried just outright removing the EEPROM and printing a test page…the printer lost its serial number, calibration information, page count, MAC address, etc, but the watermark pattern was still there…which indicates that the watermark information is burned into a level much deeper than I had originally suspected in the printer’s core. Yowza! Back to the drawing board…something to sleep on.

For those joining late into this thread, you can read more about the color printer watermarks at this [EFF webpage](http://www.eff.org/Privacy/printers/).
