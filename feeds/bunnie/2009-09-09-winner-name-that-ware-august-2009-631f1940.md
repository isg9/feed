---
title: Winner, Name that Ware August 2009
url: https://www.bunniestudios.com/blog/2009/winner-name-that-ware-august-2009/
published: "2009-09-09T10:22:36Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=512
---

# Winner, Name that Ware August 2009

The Ware for August 2009 is the [Viking Lander](http://en.wikipedia.org/wiki/Viking_program) Command Detector Board (VCLD). The Viking Landers were robotic probes sent to Mars in the 1970’s. The board itself dates from 1970.

The Ware was submitted by Norman Yarvin, and fortunately, the original author of the report on the circuit board, Bob Schuster, was in the loop to explain what it did, and how it was made.

![](http://bunniestudios.com/blog/images/ntw_aug_2009_solution.jpg)

Some notes from Bob Schuster:

> The big round thing in the lower right hand corner was an xtal used in the bit timing oscillator. The larger empty rectangular space on the lower left was for a voltage controlled xtal oscillator used in the subcarrier recovery circuits. Most of the components used were already 5 or more years in vintage, there were “better” parts, but we were using only established reliability parts.

A very detailed, and quite interesting, report summary was written by Bob, that explains the design approach behind the board and how it worked. Here’s an excerpt:

> This board is one of two boards (primary & redundant) and was part of the Command Control Unit (CCU) a box that also contained power supplies and antenna control electronics-plus other stuff which I forget. It was flown on the Viking mission to Mars and was the first USA landing of a spacecraft on another planet. Its mission was imaging and the search for evidence of life.
>
> NASA’s web site should be visited for more information about Viking.
>
> Since its mission included a search for evidence of life all flight hardware on the Lander had to be sterilized in an oven at 254 degrees F (123deg C) for two days before it could be launched. It still had to function after the sterilization of course.
>
> The board shown in the picture is the first Engineering Test Model (ETM) produced and that is why there are so many cuts and jumpers visible. The dust evident in the pictures (not very sterile) is from sitting around my attic these many years. I should have made it prettier but this was done on the spur of the moment.
>
> The purpose of the board was to recover –- reliably — the digital data being up-linked to the Lander computer from earth. The computer could then be re-programmed as conditions changed or as mission scientist came up with new things to try. The emphasis on reliability was due to the low bit rate viz 4bits/sec and the limited time available each earth day to send reprogramming commands plus the long transit times for the radio signals to reach Mars (maybe about 3 to 4 sun earth transit times or about 4x9min=36 minutes ONE WAY. If the computer data had more than one error it could not correct it, or if the detector declared a poor subcarrier signal or bad bit timing the Lander was programmed to request a retransmission of the data from earth. This event triples the time to get data uploaded to about 1.8 hours! A bit error rate of 1bit in every 100000 bits was selected by systems engineering as an error rate the mission could live with. This would presumably keep code retransmission to a minimum thereby not burning up precious contact time. Given the available transmission power of the Goldstone Tracking station and the vast distances involved the S/N ratio was just sufficient to get to the error rate specified. If the error rate was not achieved then retransmissions episodes would get too numerous and the mission would be compromised. So the VLCD had to make maximium use of the available S/N and not degrade the signal in the data recovery process.

It’s amazing this was done in the early 70’s — years before PCs were introduced — and it’s advanced communications research similar to this that laid the foundations of the mobile phone networks that we take for granted today.

You can read the full summary report [here](http://bunniestudios.com/blog/images/viking_lander_command_detector_board.pdf) (small — 40 kB). For those interested in the original history, I also have a PDF of some scans of the [original pages](http://bunniestudios.com/blog/images/vlcd_report.pdf) (large — 3 MB) of the report typed up in 1971.

Oh, and the winner — a bit hard to pick since there were a lot of thoughtful comments; I’ll go with Wang-Lo. Even though I’m not quite sure the coax cables were used as part of a delay line differentiator, he did posit a guess that this was for the Viking spacecraft. Congrats, email me for your prize.

![](http://bunniestudios.com/blog/images/ntw_aug_2009_solution2.jpg)
