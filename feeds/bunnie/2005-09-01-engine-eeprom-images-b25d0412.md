---
title: Engine EEPROM Images
url: https://www.bunniestudios.com/blog/2005/engine-eeprom-images/
published: "2005-09-01T10:07:53Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=56
---

# Engine EEPROM Images

As I was writing this, I just realized the irony that this printer I am hacking is called the [2600](http://www.2600.com/) N. First friday of September is coming up soon, hopefully I’ll be able to stop by the local chapter meeting at Regent’s Pizzeria.

Anyways, I managed to recover the EEPROM image out of the 2600N. It changes every page printed, so I have included three copies of the image here, on successive page prints:

```
00000000  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000010  00 00 00 04 00 94 2C 9B 34 9B 00 00 00 00 00 00        , 4
00000020  00 00 F1 27 5C 1B 5C 62 00 00 84 00 30 0C 00 00     '\ \b    0
00000030  0B 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
00000040  72 06 00 00 00 00 0E 00 0F 00 1D 00 2E 01 50 E4  r           . P
00000050  60 F1 C6 0F C9 00 52 04 1D 1B E7 02 DD 02 2C 9B  `     R       ,
00000060  34 9B 27 38 56 00 00 00 00 00 00 00 00 00 00 00  4 '8V
00000070  0D 00 00 00 00 00 00 00 00 00 58 04 FF FF FF FF            X
00000080  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000090  00 00 00 04 00 93 2C 9B 34 9B 00 00 00 00 00 00        , 4
000000A0  00 00 F1 27 5C 1B 5C 62 00 00 84 00 2F 0C 00 00     '\ \b    /
000000B0  0B 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
000000C0  72 06 00 00 00 00 0E 00 0F 00 1D 00 2E 01 50 E4  r           . P
000000D0  60 F1 C6 0F C9 00 52 04 00 00 98 96 C8 7F C5 84  `     R      
000000E0  1C 4A C4 CB C2 DC 64 14 CB FE 5F A1 19 65 D5 A1   J    d   _  e
000000F0  99 4B D4 CE A8 31 1C 34 8F 33 F7 10 FF FF FF FF   K   1 4 3

```

[binary form of above](http://bunniestudios.com/blog/images/2600eeprom.bin)

```
00000000  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000010  00 00 00 04 00 01 2C 9B 34 9B 00 00 00 00 00 00        , 4
00000020  00 00 F1 27 5C 1B 5C 62 00 00 83 00 9C 0B 00 00     '\ \b
00000030  0C 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
00000040  73 06 00 00 00 00 0F 00 10 00 1F 00 2E 01 50 E4  s           . P
00000050  60 F1 C6 0F C9 00 52 04 1D 1B E7 02 DD 02 2C 9B  `     R       ,
00000060  34 9B 27 38 56 00 00 00 00 00 00 00 00 00 00 00  4 '8V
00000070  0D 00 00 00 00 00 00 00 00 00 58 04 FF FF FF FF            X
00000080  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000090  00 00 00 04 00 00 2C 9B 34 9B 00 00 00 00 00 00        , 4
000000A0  00 00 F1 27 5C 1B 5C 62 00 00 83 00 9B 0B 00 00     '\ \b
000000B0  0C 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
000000C0  73 06 00 00 00 00 0F 00 10 00 1F 00 2E 01 50 E4  s           . P
000000D0  60 F1 C6 0F C9 00 52 04 00 00 98 96 C8 7F C5 84  `     R      
000000E0  1C 4A C4 CB C2 DC 64 14 CB FE 5F A1 19 65 D5 A1   J    d   _  e
000000F0  99 4B D4 CE A8 31 1C 34 8F 33 F7 10 FF FF FF FF   K   1 4 3

```

[binary form of above](http://bunniestudios.com/blog/images/2600eeprom_p2.bin)

```
00000000  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000010  00 00 00 04 00 00 2C 9B 34 9B 00 00 00 00 00 00        , 4
00000020  00 00 F1 27 5C 1B 5C 62 00 00 83 00 9B 0B 00 00     '\ \b
00000030  0D 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
00000040  74 06 00 00 00 00 0F 00 10 00 1F 00 2E 01 50 E4  t           . P
00000050  60 F1 C6 0F C9 00 52 04 1D 1B E7 02 DD 02 2C 9B  `     R       ,
00000060  34 9B 27 38 56 00 00 00 00 00 00 00 00 00 00 00  4 '8V
00000070  0D 00 00 00 00 00 00 00 00 00 58 04 FF FF FF FF            X
00000080  38 28 90 50 B0 78 3B 28 76 50 B1 78 AC 99 9A 98  8( P x;(vP x
00000090  00 00 00 04 00 00 2C 9B 34 9B 00 00 00 00 00 00        , 4
000000A0  00 00 F1 27 5C 1B 5C 62 00 00 83 00 9B 0B 00 00     '\ \b
000000B0  0D 00 BE 00 BD BF BC BE 11 22 CF 40 16 43 F3 25           " @ C %
000000C0  74 06 00 00 00 00 0F 00 10 00 1F 00 2E 01 50 E4  t           . P
000000D0  60 F1 C6 0F C9 00 52 04 00 00 98 96 C8 7F C5 84  `     R      
000000E0  1C 4A C4 CB C2 DC 64 14 CB FE 5F A1 19 65 D5 A1   J    d   _  e
000000F0  99 4B D4 CE A8 31 1C 34 8F 33 F7 10 FF FF FF FF   K   1 4 3

```

[binary form of above](http://bunniestudios.com/blog/images/2600eeprom_p3.bin)

Ah! Important note to readers: this ROM was read out using the 16-bit read mode of the EEPROM, which assumes a certain word ordering that may or may not be matched to the internal word ordering of the printer.

Getting these EEPROM images took a bit longer than I thought it would, mostly because it took me a while to realize that the SOIC pinouts were rotated relative to the DIP format used by my programmer. No matter, I managed to get everything going within the space of two CDs. Ahh…nothing like a couple [Essential Mixes](http://www.bbc.co.uk/radio1/dance/essentialmix/) to keep pace while working. I highly recommend the [Stanton Warriors 7/25/04 mix](http://www.bbc.co.uk/radio1/dance/essentialmix/the_essential_mix_archive_shtml.shtml?20040725). Too good!

I guess the question is–if the watermark is in here, where is it? I’m guessing that a large portion of the data in here corresponds to calibration fields. When I first turned on the 2600N, it spent about five minutes calibrating itself. Presumably there is a page print counter field as well, and perhaps some fields tracking other consumables. Hmmm…what to do next. A lot of possibilities, many of them very labor intensive. Time to sleep on it.

![](http://bunniestudios.com/blog/images/2600_probed.jpg)

Ugly! That’s an SOIC-to-DIP adapter I soldered in place that I got from [Emulation Technologies](http://www.emulation.com). The mess of clips is to deal with the pinout rotation from SOIC to DIP that I hadn’t anticipated. I should remember to always read the datasheet, carefully!
