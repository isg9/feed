---
title: HP2600 serial numbers
url: https://www.bunniestudios.com/blog/2005/hp2600-serial-numbers/
published: "2005-11-02T09:53:46Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=66
---

# HP2600 serial numbers

Well, it’s been a while since I’ve had a little free time to look at removing watermarks from my printer. Such is the life cycle of a spare time project.

Patrick Murphy did some excellent analysis on the engine board EEPROM, and discovered things such as [regular checksum bits](http://www.bunniestudios.com/wordpress/?p=57#comments) and so forth. For those who do not know, Patrick did a lot of the work for the [DocuColor tracking dot decoding guide](http://www.eff.org/Privacy/printers/docucolor/) on the EFF’s website.

Long story short, after poking at the engine board EEPROM for a while, he guessed there was no serial number there. It turns out that he is correct. The engine board EEPROM contained only calibration data. So, I soldered the original EEPROM back on and took off the front-end board’s EEPROM. And lo! Here’s the top of the contents of the front-end board’s 64kbit EEPROM:

```
00000000: fdb8 0000 0001 0000 0000 0000 0000 0043  ...............C
00000010: 4e42 4335 354c 3051 4a00 0000 03f0 2e17  NBC55L0QJ.......
00000020: 0409 4639 3232 314d 3900 0000 0000 0000  ..F9221M9.......
00000030: 0000 0048 6577 6c65 7474 2d50 6163 6b61  ...Hewlett-Packa
00000040: 7264 0000 0000 0000 0000 0000 4850 2043  rd..........HP C
00000050: 6f6c 6f72 204c 6173 6572 4a65 7420 3236  olor LaserJet 26
00000060: 3030 6e00 0000 0000 0000 0000 0000 4850  00n...........HP
00000070: 2043 6f6c 6f72 204c 6173 6572 4a65 7420   Color LaserJet
00000080: 3236 3030 6e00 0000 0000 0000 0000 0000  2600n...........

```

Pretty cool, huh? The serial number and a bunch of other data is stored there in plaintext. I’ve been wrong before and I’ll be wrong again. So much to learn, so much to do!

It looks like there is a bit pattern stored in the ROM a bit further down as well that might be the watermark:

```
00000110: 0000 0000 0000 0000 0000 0000 0100 0000  ................
00000120: 0114 0000 0000 0000 0100 0000 0200 0001  ................
00000130: 0200 0001 0400 0001 0500 0001 0600 0001  ................
00000140: 0900 0001 0b00 0001 0c00 0001 0d00 0001  ................
00000150: 0e00 0001 1400 0001 1500 0001 1600 0002  ................
00000160: 0000 0002 0100 0002 0200 0002 0300 0002  ................
00000170: 0400 0000 0000 0000 0100 0000 0200 0001  ................
00000180: 0200 0000 0100 0001 0500 0001 0600 0001  ................
00000190: 0900 0001 0b00 0001 0c00 0001 0d00 0001  ................
000001a0: 0e00 0001 1400 0001 1500 0001 1600 0000  ................
000001b0: 0100 0000 0100 0000 0100 0000 0100 0000  ................

```

Hmm, even further down there looks to be some semi-regular binary codes. I hope those aren’t checksums. 0x603-0x1FFF is all 0’s, so it’s a pretty empty device. I’ll tweak with it a bit over the next month or so, still pretty busy these days, unfortunately.
