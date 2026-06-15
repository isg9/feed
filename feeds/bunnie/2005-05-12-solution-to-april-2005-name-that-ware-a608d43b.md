---
title: Solution to April 2005 Name That Ware!
url: https://www.bunniestudios.com/blog/2005/solution-to-april-2005-name-that-ware/
published: "2005-05-12T08:28:11Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=18
---

# Solution to April 2005 Name That Ware!

Wow! It was tough judging the responses, some of them are pretty close.

First, let’s start with the solution.

This board is the control board from a [Zebra 310i color card printer](http://www.zebracard.com/printers/p310i.htm), the kind used to print ID badges and so forth. People who go skiing at Mammoth Mountain would notice a bank of these card printers in the back for printing the annual ski passes; they are often used by schools and businesses to print ID cards. There’s an option in this family for doing magnetic encoding of the stripes, but this particular model doesn’t feature it. This system is particularly interesting because it uses a unique security system to identify RFID-enabled ink ribbons. Now, don’t ask questions I can’t answer.

A breakdown of the board contents (correctly identified by many contestants!) is shown below (click for high-resolution (big!) version):

[![](http://bunniestudios.com/blog/images/solution_apr2005.gif)](http://bunniestudios.com/blog/images/solution_apr2005.pdf)

The winner **s** this time are Edwin Yap and DavidR. I’ll email you to get your prize preference this weekend (or email me first, emails seem to be bouncing)! [hb](http://sidekickin.blogspot.com/) pointed me at [cafepress.com](http://www.cafepress.com) and I created a line of for-fun [logo merchandise](http://www.cafepress.com/bunniestudios). I don’t make any money off of it when you buy stuff there, but they do make for more interesting give-aways to contest winners.

I chose two winners this time because Edwin had an excellent early response, but I think DavidR’s exposition and guess was probably a bit more accurate.

Edwin was right on with his note about “Heavy presense of latches hint to me that it might be sampling I/O from offboard sensors, probably from the ribbon cable headers on the right and bottom.” Also, his immediate attention to the RS232 comms port is great–the unconnected 5-pin header screams backdoor.

DavidR also insightfully noted the RS232 level converter, and he also had some good comments about the board physical gestalt: “The board isn’t very dense. PCB cost wasn’t an issue for these guys

or their form factor was determined by external considerations”. I’m guessing that’s what ruled out the automotive controller possibility for him. His guess of an instrument controller for 1DOF motion (rotation/linear motion) is so right on; the device in operation shuttles cards through a print head, and it has a number of rotational and linear systems with 1DOF.

Congratulations, and thanks for playing everyone! May’s contest will be up in a few moments, and it should be a much easier one.
