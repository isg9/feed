---
title: 'Circuit Stickers Manufacturing Retrospective: From Campaign to First Shipment'
url: https://www.bunniestudios.com/blog/2014/circuit-stickers-manufacturing-retrospective-from-campaign-to-first-shipment/
published: "2014-04-29T11:01:04Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3797
---

# Circuit Stickers Manufacturing Retrospective: From Campaign to First Shipment

Last December, Jie Qi and I launched a [crowdfunding campaign](http://www.crowdsupply.com/chibitronics/circuit-stickers) to bring circuit stickers under the brand name of “ [chibitronics](http://www.crowdsupply.com/chibitronics/circuit-stickers)” to the world.

Our [original timeline](http://www.crowdsupply.com/chibitronics/circuit-stickers/timeline) stated we would have orders shipped to Crowd Supply for fulfillment by May 2014. We’re really pleased that we were able to meet our goal, right on time, with the first shipment of over a thousand starter kits leaving the factory last week. 62 cartons of goods have cleared export in Hong Kong airport, and a second round of boxes are due to leave our factory around May 5, meaning we’ve got a really good chance of delivering product to backers by Mid-May.

![](http://bunniefoo.com/chibi/chibi-cartons.jpg)

Above: 62 cartons containing over a thousand chibitronics starter kits waiting for pickup.

**Why On-Time Delivery Is So Important**

A personal challenge of mine was to take our delivery commitment to backers very seriously. I’ve seen too many under-performing crowdfunding campaigns; I’m deeply concerned that crowdfunding for hardware is becoming synonymous with scams and spams. Kickstarter and Indiegogo have been [plagued by non-delivery](http://www.reddit.com/r/kickstarter/comments/1j6ubm/complete_list_of_funded_kickstarter_projects_that/) [and scams](http://makezine.com/2013/08/02/crowdfunding-confusion/), and their blithe [*caveat emptor* attitude](http://timeli.info/item/1386293/r_Technology_on_Reddit/After_Pando_shows_clear_evidence_of_fraud__Indiegogo_responds_by____deleting_anti_fraud_guarantee...) around campaigns is a reflection of an entrenched conflict of interest between consumers and crowdfunding websites: “hey, thanks for the nickel, but what happened to your dollar is your problem”.

I’m honestly worried that crowdfunding will get such a bad reputation that it won’t be a viable platform for well-intentioned entrepreneurs and innovators in a few years.

I made the contentious choice to go with Crowd Supply in part because they show more savvy around vetting hardware products, and their service offering to campaigns — such as fulfillment, tier-one customer support, post-campaign pre-order support, and rolling delivery dates based on demand vs. capacity — is a boon for hardware upstarts. Getting fulfillment, customer support and an ongoing e-commerce site as part of the package essentially saves me one headcount, and when your company consists of just two or three people that’s a big deal.

Crowd Supply doesn’t have the same media footprint or brand power that Kickstarter has, which means it is harder to do a big raise with them, but at the end of the day I feel it’s very important to establish an example of sustainable crowdfunding practices that is better for both the entrepreneur and the consumer. It’s not just about a money grab today: it’s about building a brand and reputation that can be trusted for years to come.

Bottom line is, if I can’t prove to current and future backers that I can deliver on-time, I stand to lose a valuable platform for launching my future products.

**On-Time Delivery Was not Easy**

We did not deliver chibitronics on time because we had it easy. When drawing up the original campaign timeline, I had a min/max bounds on delivery time spanning from just after Chinese New Year (February) to around April. I added one month beyond the max just to be safe. We ended up using every last bit of padding in the schedule.

I made a lot of mistakes along the way, and through a combination of hard work, luck, planning, and strong factory relationships, we were able to battle through many hardships. Here’s a few examples of lessons learned.

**A simple request for one is not necessarily a simple request for another.** Included with every starter kit is a [fantastic book (free to download)](http://bunniefoo.com/chibi/sketchbook-en-v1.pdf) written by Jie Qi which serves as a step-by-step, self-instruction guide to designing with circuit stickers. The book is unusual because you’re meant to paste electronic circuits into it. We had to customize several aspects of the printing, from the paper thickness (to get the right light diffusion) to the binding (for a better circuit crafting experience) to the little pocket in the back (to hold swatches of Z-tape and Linqstat material). Most of these requests were relatively easy to accommodate, but one in particular threw the printer for a loop. We needed the metal spiral binding of the book to be non-conductive, so if someone accidentally laid copper tape on the binding it wouldn’t cause a short circuit.

![](http://bunniefoo.com/chibi/chibi-book-cover.jpg)

Below is an example of how a circuit looks in the book — in this case, the DIY pressure sensor tutorial (click on image for a larger version).

[![](http://bunniefoo.com/chibi/chibi-pressuresensor_sm.jpg)](http://bunniefoo.com/chibi/chibi-pressuresensor.jpg)

Checking for conductivity of a wire seems like a simple enough request for someone who designs circuits for a living, but for a book printer, it’s extremely weird. No part of traditional book printing or binding requires such knowledge. Because of this, the original response from the printer is “we can’t guarantee anything about the conductivity of the binding wire”, and sure enough, the first sample was non-conductive, but the second was conductive and they could not explain why. This is where face to face meetings are invaluable. Instead of yelling at them over email, we arranged a meeting with the vendor during one of my monthly trips to Shenzhen. We had a productive discussion about their concerns, and at the conclusion of the meeting we ordered them a $5 multimeter in exchange for a guarantee of a non-conductive book spine. In the end, the vendor was simply unwilling to guarantee something for which he had no quality control procedure — an extremely reasonable position — and we just had to educate the vendor on how to use a multimeter.

To wit, this unusual non-conductivity requirement did extend our lead time by several days and added a few cents to the cost of the book, but overall, I’m willing to accept that compromise.

**Never skip a checkplot.** I alluded to this poignant lesson with the following tweet:

> Skip a gerber checkplot … scrap 200+ PCBs due to config error on just one pad's soldermask. Lose four weeks and $,$$$ [#fml](https://twitter.com/search?q=%23fml&src=hash) [#hardwareishard](https://twitter.com/search?q=%23hardwareishard&src=hash)
>
> — bunnie (@bunniestudios) [January 15, 2014](https://twitter.com/bunniestudios/statuses/423483963902943232)

The pad shapes for chibitronics are complex polyline geometries, which aren’t handled so gracefully by Altium. One problem I’ve discovered the hard way is the soldermask layer occasionally disappears for pads with complex geometry. One version of the file will have a soldermask opening, and in the next save checkpoint, it’s gone. This sort of bug is rare, but it does happen. Normally I do a gerber re-import check with a third-party tool, but since this was a re-order of an existing design that worked before, and I was in a rush, I skipped the check. Result? thousands of dollars of PCBs scrapped, four weeks gone from the schedule. Ouch.

Good thing I padded my delivery dates, and good thing I keep a bottle of fine scotch on hand to help bitter reminders of what happens when I get complacent go down a little bit easier.

**If something can fit in a right and a wrong way, the wrong way will happen.** I’m paranoid about this problem — I’ve been burned by it many times before. The effects sticker sheet is a prime example of this problem waiting to happen. It is an array of four otherwise identical stickers, except for the LED flashing pattern they output. The LED flashing pattern is controlled by software, and trying to manage four separate firmware files and get them all loaded into the right spot in a tester is a nightmare waiting to happen. So, I designed the stickers to all use exactly the same firmware; their behaviors set by the value of a single external resistor.

So the logic goes: if all the stickers have the same firmware, it’s impossible to have a “wrong way” to program the stickers. Right?

![](http://bunniefoo.com/chibi/chibi-effects-v1.jpg)

Unfortunately, I also designed the master PCB panels so they were perfectly symmetric. You can load the panels into the assembly robot rotated by pi radians and the assembly program runs flawlessly — except that the resistors which set the firmware behavior are populated in reverse order from the silkscreen labels. Despite having fiducial holes and text on the PCBs in both Chinese and English that are uniquely orienting, this problem actually happened. The first samples of the effects stickers were “blinking” where it said “heartbeat”, “fading” where it said “twinkle”, and vice-versa.

Fortunately, the factory very consistently loaded the boards in backwards, which is the best case for a problem like this. I rushed a firmware patch (which is in itself a risky thing to do) that reversed the interpretation of the resistor values, and had a new set of samples fedexed to me in Singapore for sanity checking. We also built a secondary test jig to add a manual double-check for correct flashing behavior on the line in China. Although, in making that additional test, we were confronted with another common problem —

**Some things just don’t translate well into Chinese**. When coming up with instructions to describe the difference between “fading” (a slow blinking pattern) and “twinkling” (a flickering pattern), it turns out that the Chinese translation for “blink” and “twinkle” are similar. Twinkle translates to 闪烁 (“flickering, twinkling”) or 闪耀 （to glint, to glitter, to sparkle), whereas blink translates to 闪闪 (“flickering, sparkling, glittering”) or 闪亮 (“brilliant, shiny, to glisten, to twinkle”). I always dread making up subjective descriptions for test operators in Chinese, which is part of the reason we try to automate as many tests as possible. As one of my Chinese friends once quipped, Mandarin is a wonderful language for poetry and arts, but difficult for precise technical communications.

Above is an example of the effects stickers in action. How does one come up with a bulletproof, cross-cultural explanation of the difference between fading (on the left) and twinkling (on the right), using only simple terms anyone can understand, e.g. avoiding technical terms such as random, frequency, hertz, periodic, etc.

After viewing the video, our factory recommended to use “渐变” (gradual change) for fade and “闪烁” (flickering, twinkling) for twinkle. I’m not yet convinced this is a bulletproof description, but it’s superior to any translation I could come up with.

Funny enough, it was also a challenge for Jie and I to agree upon what a “twinkle” effect should look like. We had several long conversations on the topic, followed up by demo videos to clarify the desired effect. The implementation was basically tweaking code until it “looked about right” — Jie described our first iteration of the effect as “closer to a lightning storm than twinkling”. Given the difficulty we had describing the effect to each other, it’s no surprise I’m running into challenges accurately describing the effect in Chinese.

**Eliminate single points of failure.** When we built test jigs, we built two copies of each, even though throughput requirements demanded just one. Why? Just in case one failed. And guess what, one of them failed, for reasons as of yet unknown. Thank goodness we built two copies, or I’d be in China right now trying to diagnose why our sole test jig isn’t working.

**Sometimes last minute changes are worth it.** About six weeks ago, Jie suggested that we should include a stencil with the sensor/microcontroller kits. She reasoned that it can be difficult to lay out the copper tape patterns for complex stickers, such as the microcontroller (featuring seven pads), without a drawing of the contact patterns. I originally resisted the idea — we were just weeks away from finalizing the order, and I didn’t want to delay shipment on account of something we didn’t originally promise. As Jie is discovering, I can be very temperamental, especially when it comes to things that can cause schedule slips (sorry Jie, thanks for bearing with me!). However, her arguments were sound and so I instructed our factory to search for a stencil vendor. Two weeks passed and we couldn’t find anyone willing to take the job, but our factory’s sourcing department wasn’t going to give up so easily. Eventually, they found one vendor who had enough material in stock to tool up a die cutter and turn a couple thousand stencils within two weeks — just barely in time to meet the schedule.

[![](http://bunniefoo.com/chibi/chibi-stencil_sm.jpg)](http://bunniefoo.com/chibi/chibi-stencil.jpg)

When I got samples of the sensor/micro kit with the stencils, I gave them a whirl, and Jie was absolutely right about the utility of the stencils. The user experience is vastly improved when you have a template to work from, particularly for the microcontroller sticker with seven closely spaced pads. And so, even though it wasn’t promised as part of the original campaign, all backers who ordered the sensor/micro kit are getting a free stencil to help with laying out their designs.

**Chinese New Year has a big impact the supply chain.** Even though Chinese New Year (CNY) is a 2-week holiday, [our initial schedule](http://www.crowdsupply.com/chibitronics/circuit-stickers/timeline) essentially wrote off the month of February. Reality matched this expectation, but I thought it’d be helpful to share an anecdote on exactly how CNY ended up impacting this project. We had a draft manuscript of our book in January, but I couldn’t get a complete sample until March. It’s not because the printer was off work for a month straight — their holiday, like everyone else’s, was about two weeks long. However, the paper vendor started its holiday about 10 days before the printer, and the binding vendor ended its holiday about 10 days after the printer. So even though each vendor took two weeks off, the net supply chain for printing a custom book was out for holiday for around 24 days — effectively the entire month of February. The staggered observance of CNY is necessary because of the [sheer magnitude of human migration](http://news.nationalgeographic.com/news/2014/01/140131-lunar-new-year-china-migration-baidu-map/) that accompanies the holiday.

**Shipping is expensive, and difficult.** When I ran the initial numbers on shipping, one thing I realized is we weren’t selling circuit stickers — at least by volume and weight, our principle product is printed paper (the book). So, to optimize logistics cost, I was pushing to ship starter kits (which contain a book) and additional stand-alone book orders by ocean, rather than air.

We actually had starter kits and books ready to go almost four weeks ago, but we just couldn’t get a reasonable quotation for the cost of shipping them by ocean. We spent almost three weeks haggling and quoting with ocean freight companies, and in the end, their price was basically the same as going by air, but would take three weeks longer and incurred more risk. It turns out that freight cost is a minor component of going by ocean, and you get killed by a multitude of surcharges, from paying the longshoreman to paying all the intermediate warehouses and brokers that handle your goods at the dock. All these fixed costs add up, such that even though we were shipping over 60 cartons of goods, air shipping was still a cost-effective option. To wit, a Maersk 40′ sea container will fit over 1250 cartons each containing 40 starter kits, so we’re still an order of magnitude away from being able to efficiently utilize ocean freight.

**We’re not out of the Woods Yet.** However excited I am about this milestone, I have to remind myself not to count my chickens before they hatch. Problems ranging from a routine screw-up by UPS to a tragic aviation accident to a logistics problem at Crowd Supply’s fulfillment depot to a customs problem could stymie an on-time delivery.

But, at the very least, at this point we can say we’ve done everything reasonably within our power to deliver on-time.

We are looking forward to hearing our backer’s feedback on chibitronics. If you are curious and want to join in on the fun, the [Crowd Supply site is taking orders](http://www.crowdsupply.com/chibitronics/circuit-stickers), and Jie and I will be at [Maker Faire Bay Area 2014](http://makerfaire.com/makers/circuit-stickers/), in the Expo hall, teaching free workshops on how to learn and play with circuit stickers. We’re looking forward to meeting you!
