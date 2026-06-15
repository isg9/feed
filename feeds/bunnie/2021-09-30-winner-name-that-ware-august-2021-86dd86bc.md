---
title: Winner, Name that Ware August 2021
url: https://www.bunniestudios.com/blog/2021/winner-name-that-ware-august-2021/
published: "2021-09-30T15:38:18Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=6242
---

# Winner, Name that Ware August 2021

The Ware for August 2021 is a DirectTV receiver model D10. As I had surmised, it was quite easy to guess the nature of the ware, based on obvious clues such as the smart card slot, tuner circuit, and the general TV-receiver-esque vibe about the board. One does not integrate a line voltage power supply and an RF front end into a single circuit board unless one is sure to sell millions of them; the regulatory overhead is just too great otherwise.

Anyways, to make the ware a bit more challenging I had asked for comments specifically about the unpopulated cut-out section of the board. Participants correctly recognized from the get-go that it should be an accessory to the receiver, likely related to a remote control. GotNoTime even went so far as to call out a part number, the KESRX01 ASK receiver chip (as listed in the schematics that were previously linked by Eben Olson). While the schematics share the chip designator U33101, unfortunately, the KESRX01 is a 24-pin package, but the board has a 28-pin footprint. Thus, it’s quite likely these are closely related schematics, probably made by the same OEM, but they also cannot be an exact match.

This motivated me to poke a bit more; a bit of googling around revealed a good match for the footprint and functional category: the TDA5200. Below is a juxtaposition of the reference schematic from the datasheet along with the layout:

[![](https://bunniefoo.com/ntw/tda5200_sm.png)](https://bunniefoo.com/ntw/tda5200.png)

And just for quick reference, here’s the pinout and internal block diagram of the chip:

![](https://bunniefoo.com/ntw/tda5200_pins.png)

![](https://bunniefoo.com/ntw/tda5200_block.png)

I got as far as confirming that the power supply lines match up, and that the crystal and RF lines seem to make sense, which means it’s likely to be this chip or at least a closely related one. If I had to build up a functional circuit using this board blank and no schematics, I’d say it’s likely to succeed, modulo a couple patch wires and a bit of antenna tuning.

My propensity to figure out what goes into the blank spots of a circuit board is a quirk that stems from having to, at times, reverse engineer devices that have had the part numbers sanded off or obscured for “security” reasons. That’s probably part of the reason I’m drawn toward empty spots on circuit boards; they are good practice for looking at context and circuit traces to figure out part numbers. It’s like circuit board karaoke!

I guess this month there isn’t a winner, since nobody *quite* got the answer I was looking for, so I’ll leave it at that (although I am probably partially to blame for not being clearer as to what I was hoping to see as an answer!). However, thanks again to everyone for playing!
