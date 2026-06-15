---
title: Winner, Name That Ware January 2010
url: https://www.bunniestudios.com/blog/2010/winner-name-that-ware-2010/
published: "2010-02-16T08:31:55Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=898
---

# Winner, Name That Ware January 2010

The Ware for January 2010 was a [2Gbyte microSD card](http://www.amazon.com/gp/product/B000VOU91U?ie=UTF8&tag=bunniestudios-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=B000VOU91U)![](http://www.assoc-amazon.com/e/ir?t=bunniestudios-20&l=as2&o=1&a=B000VOU91U). Below is an image of the whole card, with the area that was shown in the contest photo circled in black.

![](http://bunniestudios.com/blog/images/ntw_jan_10_winner.jpg)

The region imaged for the contest is about 1/8th to 1/16th of the whole card; with a capacity of 16 billion bits, the imaged region of the FLASH chip alone is therefore showing at least a billion transistors. The lighter-colored region in the center of the FLASH chip is where I nicked the die prior to acid etching as I was grinding the package. I’ve since learned that grinding is unnecessary for microSD cards. :-)

[![](http://imgs.xkcd.com/comics/microsd.png)](http://xkcd.com/691/)

MicroSD cards are wholly amazing devices ( [XKCD](http://xkcd.com/) seems to share my sentiment on this), not simply because of their great storage to size ratio, but also because of their internal construction. I was a little surprised by two things: first, the use of stacked CSP (chip scale package) construction, and second, the sheer size of the FLASH die.

To do the CSP in such a thin package, both die (the FLASH and the controller) have to be thinned. Before each wafer is diced, they are flipped over and mechanically polished. The controller die in particular looks to be polished to a thickness of about 0.1mm — near the width of a human hair.

For the FLASH die, I was a bit surprised by how large it was. Before decapsulating the card, I was expecting to see one of two things: perhaps two die side by side, with the memory chip maybe about 6-7mm on a side and a 3mm/edge controller chip next to it, or a single fully-integrated FLASH + controller die. Instead, the memory chip is about 9 x 13mm — filling nearly the entire package edge to edge — and the controller chip is a sliver of silicon only 1.5mm wide and a shy more than 4mm long. Also, the actual FLASH chip die is simply a [K9GAG08U0D](http://www.samsung.com/global/business/semiconductor/productList.do?fmly_id=672), which is a common MLC device that you can also purchase in a TSOP package.

![](http://bunniestudios.com/blog/images/ntw_jan_10_k9ga.jpg)

This is pretty clever on Samsung’s part. By sizing their commodity street FLASH to fit inside a microSD card, they are able to feather demand between popular SKUs. It’s also amusing that a full 32-bit ARM core is used as the controller between the FLASH device and the SD card interface. The world is just crawling with ARM controllers…

As for the winner, a bit tough to pick this time. A lot of people walked around the answer, thrown off by the CMP fill tiles, and assuming the device had something to do with a secure system.

The metal squares, while they do obscure the circuitry beneath them, are not active elements nor are they put there for the purpose of security. They are there actually to ensure that the fill density of metal versus dielectric is within a certain ratio. This is due to a requirement of a process known as CMP (chemical-mechanical polishing), where a chip’s active surface is planarized, layer by layer, by rubbing abrasive pads with a fine-grit chemical slurry across the surface. If the fill ratio is not obeyed, then portions outside the ratio will either polish too quickly or too slowly due to the differential in hardness between the metal and the oxide dielectric, and the result will be a non-planar surface.

![](http://bunniestudios.com/blog/images/ntw_jan_10_cmp.jpg)

You can see the fill density equalization going on in the picture above. The very large silvery tiles are from a very coarse top layer metal used just for bonding pads. The copper-hued tiles are from a lower wiring layer. You can see just a few of those wires in this photo. The area immediately around the wire has no tiling, but the otherwise open expanse is entirely filled in to ensure that the sparse, tiny wires don’t get polished away during the CMP operation.

Thus, when you see CMP tiling you know you are dealing with a deep submicron process with a lot of metal layers. Also, active protection meshes typically do not use unconnected pieces of metal; instead, the metal is connected in a series chain so damage to any individual element is easily detected by the monitoring circuit.

Since MegabytePhreak is the first answer to use the term “microSD” explicitly, he/she is the winner (by just 3 hours over Burlap). Congrats! email me for your prize.
