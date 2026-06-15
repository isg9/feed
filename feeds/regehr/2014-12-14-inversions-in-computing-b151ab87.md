---
title: Inversions in Computing
url: https://blog.regehr.org/archives/1204
published: "2014-12-14T23:44:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1204
---

# Inversions in Computing

Some computer things change very slowly; for example, my newish desktop at home has a PS/2 port. Other things change rapidly: my 2010 iPad is kind of a stone-age relic now. This kind of differential progress creates some funny inversions. A couple of historical examples:

- Apparently at one point in the 80s or 90s (this isn’t a firsthand story– I’d appreciate recollections or citations) the processor available in an Apple printer was so fast that people would offload numerical computations to their printers.

- I spent the summer of 1997 working for [Myricom](http://www.myricom.com/). Using the then-current Pentium Pro machines, you could move data between two computers faster than you could do a local memcpy(). I’m pretty sure there was something wrong with the chipset for these processors, causing especially poor memcpy() performance, but I’ve lost the details.

What are the modern examples? A few come to mind:

- Non-volatile storage used to be really slow because disk reads involve waiting for moving parts. Huge increases in the speed of non-volatile storage have lead Intel to provide [instruction set support for non-volatile memory](http://danluu.com/clwb-pcommit/).

- I realize the example may not be 100% serious, but placing a [filesystem in video RAM](https://github.com/Overv/vramfs) represents some kind of serious inversion.

- A device that [looks like a flash disk but includes a Linux machine](https://www.crowdsupply.com/inverse-path/usb-armory) that would have seemed pretty fast a few years ago.

- Running [C++ in the browser by compiling to asm.js](https://github.com/kripken/emscripten).

Anyhow, I enjoy computing inversions since they challenge our assumptions.
