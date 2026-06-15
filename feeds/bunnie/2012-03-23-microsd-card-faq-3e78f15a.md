---
title: MicroSD card FAQ
url: https://www.bunniestudios.com/blog/2012/microsd-card-faq/
published: "2012-03-23T08:05:45Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=2297
---

# MicroSD card FAQ

A while back I wrote [an analysis of fake microSD](http://www.bunniestudios.com/blog/?page_id=1022) cards. As a result of the post, I’ve received this question regularly via email:

“I’m trying to buy a thousand microSD cards for my embedded controller project. How do you qualify a microSD card?”

So, I thought it might be helpful to share my answer here.

There’s this awkward phase between the weekend project (where you buy your microSD card from Best Buy for $20 and have a no-questions return policy) and being Nokia (where you buy the same cards for $2 in quantities large enough to actually have leverage over vendors). When you source a few thousand cards at a time on the wholesale spot market, you’re basically on your own to control quality.

As far as process control, some vendors are easier to work with than others. Samsung will bump their part numbers based on die revs or other significant internal changes to the card. Sandisk, on the other hand, uses a very short part number for their cards, so you have no idea if the NAND on the inside is MLC or TLC, etc.; you just know the capacity and the card is simply guaranteed to perform to spec. To wit, Sandisk is very thorough about ensuring they meet the spec. However, it’s the edge cases that usually bite you in production; regardless of the spec, every die/controller combo has some character and your embedded controller may bring out some of that color. And, of course, there’s the fakes — Sandisk is a huge target for fakes, people who want to borrow their good name to sell you a batch of shoddy cards.

If you’re working with a distributor, get a copy of their authorization letter that certifies the relationship with the brand they are selling. It’s easy to fake the certificate, but it’s a good formality to pursue anyways. If you can, get the upstream brand to confirm the distribution relationship.

Aside from these supply-chain side things, here’s a check-list of technical tests to run on your cards:

For each new distributor:

1\. I read out the CID and CSD registers and decode them. This is easy to do on linux with a directly connected microSD card. You cannot do this if the card is plugged into a USB adapter — you need to have the card plugged into a direct SD interface. The CID and CSD should look “right” i.e., the manufacturer ID should make sense (unfortunately the manufacture ID codes are all secret, but I can assure you it’s not supposed to be FF or 00), serial numbers should be some big number, date codes correct, etc.

2\. Do a “full write” test at least once. i.e., create a random block of data that’s the putative size of the card, and dd it into the card. Then, do an md5sum of the contents of the card. This will identify loopback tricks that fake capacity. This is a relatively common trick that is surprisingly hard to detect, because many cards are only used to less than 50% capacity in real life.

3\. Do a reboot test, to understand the behavior of the controller/die combo during ungraceful powerdown. It’s less important on systems that can never have their battery removed.

Before the test, I do a recursive find piped to md5sum to get a full map of all the files in the card. Then, I use a script that writes a random amount of /dev/urandom data in odd-sized blocks (ranging from a couple hundred bytes to a couple megabytes) to the card and then calls sync, in a constant loop after boot. For each block written, the md5sum is recorded. At boot time, all old blocks are checked for md5sum consistency and then deleted. The system under test is automatically power cycled by cutting the AC power about once very 2-3 minutes plus some random interval (depends on how long it takes your device to boot). I cut on the AC power side to capture the effects of the power decay curve of the wall adapter; the logic goes that a clean power down is less likely to cause problems than a gradual powerdown. I run the test on a cohort of at least 2 systems for 2 days straight. If you want to get fancy, you have the system upload its statistics to a server so you can see exactly when it starts to fail. After a couple of days, I extract the card from the system and redo the recursive find with md5sum to verify that no non-critical files have been corrupted that would be difficult to notice without the comprehensive check. Be sure, of course, to ignore files that naturally vary.

I still don’t have a straight answer on why some cards perform better under this test and others fail miserably. Ultimately, however, every card I’ve encountered eventually corrupts the filesystem after enough cycles, it’s just a matter of how long. I feel comfortable if I can reliably get to ten thousand ungraceful reboots-while-writing before failure. Note that supposedly eMMC has design features that harden cards against these problems, but I’ve never had the luxury of building such high volume systems that eMMC becomes an affordable option. Besides, I consider giving users the ability to remove the firmware card and reflash it with new code using a common USB adapter an important feature, at least in the systems I design. Mobile phone carriers would think differently.

Of course, once a vendor is qualified, they can still send you bad lots.

For each new lot I get, I take a few cards and burn them myself and check they boot the system before handing them over the factory. I also manually inspect the CID/CSD to ensure that the manufacturer’s IDs haven’t rotated and I inspect the laser markings to ensure that the lot number changes (it should — if it doesn’t then they are pulling something wonky on you). I also compare the circuit trace pattern on the back, visible through the reliefs in the solder resist coating. If you have easy access to an X-ray machine (some CMs have them on site) you can go so far as to compare the internal construction in the x-ray to see if the dies have been revved. If all these are the same you’re probably good to go on the new lot, but I do pay attention to the failure rate data in the first couple hours of production just to make sure there isn’t something to worry about.

There’s probably a bunch of other tests, techniques and good ideas that I should be aware of…look forward to reading the comments!
