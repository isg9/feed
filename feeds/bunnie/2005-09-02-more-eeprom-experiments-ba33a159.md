---
title: More EEPROM Experiments
url: https://www.bunniestudios.com/blog/2005/more-eeprom-experiments/
published: "2005-09-02T07:47:38Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=57
---

# More EEPROM Experiments

When in doubt, knock them out! I took one of the read-in EEPROM images and knocked out a few bytes here and there in chosen places to see what happens to the EEPROM. Here are the results:

Installing 00 in location 04: No effect. Printer re-wrote correct value there.

Installing 00 in location C8: No effect: Printer re-wrote correct value there.

Installing 00 in location E5: Printer reports engine error on boot.

Installing 00 in location F5: Printer reports engine error on boot.

That helps eliminate some possibilities.

DC requested that I rescan, with the improved scanner, the first page that I had from a *different* serial number printer. Here is a swatch from the rescan:

![](http://bunniestudios.com/blog/images/other_sn.jpg)

Here’s [a torrent](http://bunniestudios.com/blog/images/rescan.tif.torrent) for the image file.

Anybody in the San Diego area have an HP 2600N that I can borrow for a weekend? :-D I want your EEPROM contents!
