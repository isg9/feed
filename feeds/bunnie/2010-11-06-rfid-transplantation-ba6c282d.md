---
title: RFID Transplantation
url: https://www.bunniestudios.com/blog/2010/rfid-transplantation/
published: "2010-11-06T09:55:45Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1379
---

# RFID Transplantation

One of the nice things about living in Singapore is its comprehensive mass-transit system. The [SMRT](http://www.smrt.com.sg/trains/network_map.asp) blankets the 26-mile by 14-mile island nation with a network of 78 stations and an extensive bus system. This is in stark contrast to San Diego county, which is over 15 times the land area in size but has only 2/3rds the population and is covered by a trolley system with 53 stops. Needless to say, it’s impossible to live in San Diego without a car; while driving is a privilege, it’s a burden when you are *required* to do it. So, I’m quite happy now to have the option of taking the SMRT, safely answering emails and playing video games while the train takes me to my destination.

However, one small irritation I’ve encountered with the SMRT is that the “ [EZlink](http://en.wikipedia.org/wiki/EZ-Link)” RFID card system used in Singapore conflicts with the two other RFID subway cards in my wallet (the [Shenzhen Tong](http://en.wikipedia.org/wiki/Shenzhen_Tong) and the [Hong Kong Octopus](http://en.wikipedia.org/wiki/Octopus_card) card), so as I pass through the busy turnstyles, about half the time I get an invalid card error, causing much irritation among the people behind me as I sort through my RFID card collection to pick out the EZlink card.

Having seen Japan’s [Suica](http://en.wikipedia.org/wiki/Suica) system integrated into mobile phones, I thought, why not stick the EZlink chip inside my phone? Since the EZlink card also serves as a payment card, I can get around the city with nothing but my phone, buying beverages at 7-11, and paying taxi, bus and train fares while texting my buddies without carrying a scrap of cash.

As a general note, transplanting RFID chips is a much cleaner solution from both the legal and technical perspective versus cracking the security and programming your own RFID to be compatible with the existing payment system. While many of the security systems used in RFID are already broken or have serious known vulnerabilities, I can’t think of any country where the authorities would take kindly to you doing it. And, while the 3DES system used in the EZlink’s security isn’t the strongest out there, it’s still hard enough to crack that it’s just not worth the effort.

Transplanting the RFID chip ended up taking only a couple hours in the end; I think it’s a handy enough hack that I’m sharing the details on how to do it. Unfortunately, few of my American readers would have an immediate use for it, since RFID payment and subway transportation technology really hasn’t reached most of the US population…I must remark at this point that living overseas really highlights how behind the US is in some areas. I have [100 Mbit](http://www.starhub.com/broadband/athome.html#tabcontrol-1) broadband service in my home for about US$60/month…and just a couple months ago they rolled out 1 Gbit fiber-to-the-home in my neighborhood, and I’m tempted to upgrade, although I’m not quite sure what, exactly, I’d do with a Gigabit connection. It’s also sad to find in the details of my [Japanese mobile data plan](http://www.emobile.jp/en/) that the US’s 3G service is classified in the same performance tier as Africa’s 3G services.

Note to locals: I picked the EZlink card (as opposed to the competing NETS system) because they have convenient top-up kiosks in the station where you just lay the card on a pedestal to recharge it. The NETS system requires you to put the card into a slot reader, or to give it to an agent, both of which are not an option when you’ve hacked the card into a mobile phone.

![](http://bunniestudios.com/blog/images/post_ezlink0.jpg)

The EZlink card uses a 13.56 MHz contactless RFID system, so inside the card there’s an actual silicon chip, and an embedded antenna. Above is a photo of the card with the chip’s location (top right corner) revealed. The easiest way to locate the chip is to look at the reflection of a lightbulb off the surface and observe the slight bump underneath the surface where the chip is located. Outline the location with a marker and use a hobby knife to scrape away at the plastic.

![](http://bunniestudios.com/blog/images/post_ezlink1.jpg)

Scraping away at the plastic on the opposing side as well makes the chip easier to release:

![](http://bunniestudios.com/blog/images/post_ezlink2.jpg)

Lift the chip out very delicately, as there is a loop of copper wire bonded to the chip’s leadframe. If pulled too hard, the leadframe will be damaged — it must be kept intact, since an alternate antenna will be soldered to the leadframe later on. Below is a photo of the chip lifted up partially, revealing the copper wires.

![](http://bunniestudios.com/blog/images/post_ezlink6.jpg)

Below is a photo of the chip’s leadframe, with arrows pointing to the solder points on the leadframe. Notice how the metal on the left and right side are not actually electrically connected to the metal paddle in the center, thus creating three electrically isolated regions. Take caution not to short them together.

![](http://bunniestudios.com/blog/images/post_ezlink5.jpg)

Now that the chip is free, attach it to a suitable antenna. For this hack, I took a 13.56 MHz RFID bracelet and re-used the antenna from it. The bracelet is made by Precision Dynamics, a [PDC Smart Superband 470](http://www.pdcorp.com/en-us/rfid-ent/470-smart-superband-wristbands.html). You can also make your own antenna, but RFIDs are so common it may be easier to scavenge an existing antenna out of any used 13.56 MHz RFID.

![](http://bunniestudios.com/blog/images/post_rfidband.jpg)

Cutting open the band is easily done with a pair of scissors:

[![](http://bunniestudios.com/blog/images/post_rfid_before_sm.jpg)](http://bunniestudios.com/blog/images/post_rfid_before.jpg)

Next, carefully cut the existing chip out of the antenna. Since it’s all printed on thin flexible plastic, this is easily done with a hobby knife.

![](http://bunniestudios.com/blog/images/post_ezlink_rfid1.jpg)

Above is a photo of the partially-cut chip. When cutting the chip out, be sure to leave the antenna contacts on either side, as these will be used to solder to the EZlink RFID chip’s leadframe tabs. Below is a photo of the chip itself, after it has been freed of its bond to the antenna.

[![](http://bunniestudios.com/blog/images/post_rfidzoom1_sm.jpg)](http://bunniestudios.com/blog/images/post_rfidzoom1.jpg)

Next, lay some kapton tape down in the region of the RFID chip bonding area to protect the delicate antenna traces underneath. Slide the RFID chip in between the antenna contacts, and solder it down:

![](http://bunniestudios.com/blog/images/post_ezlink_hybrid1.jpg)

![](http://bunniestudios.com/blog/images/post_ezlink_hybrid2.jpg)

Soldering the chip takes a deft hand, since you’re soldering onto soft plastic that will melt if you apply too much heat. However, a bit of solder flux applied before the operation and a temperature-controlled iron set to the lowest temperature that will still melt solder makes things easier.

And that’s basically it! The final EZlink chip + grafted antenna assembly is very thin and flexible:

[![](http://bunniestudios.com/blog/images/post_ezlink_film_sm.jpg)](http://bunniestudios.com/blog/images/post_ezlink_film.jpg)

[![](http://bunniestudios.com/blog/images/post_ezlink_film2_sm.jpg)](http://bunniestudios.com/blog/images/post_ezlink_film2.jpg)

It’s thin enough to be taped inside the battery compartment of my local phone. Positioning of the antenna is important; it needs to clear the battery pack as much as possible, as the battery pack interferes with the RFID signal. Here’s a photo of the compartment with the back cover off:

![](http://bunniestudios.com/blog/images/post_ezlink_inphone.jpg)

I’m guessing the TSA would *not* be entertained if they found this on me given the recent use of mobile phones in cargo bombs…which is why I stuck it into my local-only feature phone, instead of my international-use Blackberry.

And, here it is, in my local SMRT station, showing the latest balance:

![](http://bunniestudios.com/blog/images/post_ezlink_working.jpg)

The final antenna+RFID assembly is thin and flexible enough to be hidden in a number of convenient locations; it could be put into the wristband of a watch, sewn into clothing (although, I wouldn’t put this through the wash), or integrated into jewelry.
