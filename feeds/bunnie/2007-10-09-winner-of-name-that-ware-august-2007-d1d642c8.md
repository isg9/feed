---
title: Winner of Name that Ware August 2007!
url: https://www.bunniestudios.com/blog/2007/winner-of-name-that-ware-august-2007/
published: "2007-10-09T04:33:24Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=206
---

# Winner of Name that Ware August 2007!

The Ware for August 2007 is a die shot of the EEPROM memory area from an [MF RC530 ISO 14443A Reader IC](http://www.nxp.com/acrobat_download/other/identification/m067420.pdf) by NXP. The MF RC530 is one of the RFID readers IC that can be used in the MIFARE system, and it employs the CRYPTO-1 protocol. This EEPROM memory is part of a secure memory region that contains the keys used by the CRYPTO-1 protocol. One question I am still puzzling over is the location of the CRYPTO-1 implementation on this die. CRYPTO-1 is a proprietary cipher, and some friends of mine were curious as to the algorithm–so I contacted [Flylogic](http://flylogic.net) and we popped the top of the MF RC530 to have a look around. My original opinion was that the array of gates immediately below the EEPROM was the CRYPTO-1 cipher. However, tracing the wires in and out of the block seem to disagree with this functionality, and instead it looks more appropriate as the programming and sequencing logic for the EEPROM. There is a microcode ROM on this chip as well, so quite possibly the cipher is implemented using part of the microcode ROM, or it is implemented using random, compiled logic.

Several readers correctly guessed the capacity of the memory array at 4kbits, although many were unclear as to what kind of array it was–some guessed DRAM, others SRAM, others a PLA. Actually, all of these are pretty good guesses. The clue that makes me think this is an EEPROM (or FLASH) array is the charge pumps on the right hand side. The large capacitors are used in a set of voltage doublers to create the high voltage necessary to erase and program the floating gates of the EEPROM memory devices. The high voltages are unique, at least in today’s modern processes, to FLASH and EEPROM devices. Christian Vogel basically nailed this analysis, so he is the winner. Congratulations, email me for your prize!
