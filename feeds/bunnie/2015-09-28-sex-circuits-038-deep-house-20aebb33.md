---
title: Sex, Circuits &#038; Deep House
url: https://www.bunniestudios.com/blog/2015/sex-circuits-deep-house/
published: "2015-09-28T10:16:51Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=4484
---

# Sex, Circuits &#038; Deep House

[![P9010002](https://farm6.staticflickr.com/5629/21128268819_c1ab0f5132.jpg)](https://www.flickr.com/photos/nargopolis/21128268819/in/pool-theinstitute2015/ "P9010002")

*Cari with the Institute Blinky Badge at Burning Man 2015. Photo credit: Nagutron.*

This year for Burning Man, I built a networked light badge for my theme camp, “The Institute”. Walking in the desert at night with no light is a dangerous proposition – you can get run over by cars, bikes, or twist an ankle tripping over an errant bit of rebar sticking out of the ground. Thus, the outrageous, bordering grotesque, lighting spectacle that Burning Man becomes at night grows out of a central need for safety in the dark. While a pair of dimly flashing red LEDs should be sufficient to ensure one’s safety, anything more subtle than a Las Vegas strip billboard tends to go unnoticed by fast-moving bikers thanks to the LED arms race that has become Burning Man at night.

I wanted to make a bit of lighting that my campmates could use to stay safe – and optionally stay classy by offering a range of more subtle lighting effects. I also wanted the light patterns to be individually unique, allowing easy identification in dark, dusty nights. However, diddling with knobs and code isn’t a very social experience, and few people bring laptops to Burning Man. I wanted to come up with a way for people to craft an identity that was inherently social and interactive. In an act of shameless biomimicry, I copied nature’s most popular protocol for creating individuals – sex.

By adding a peer-to-peer radio in each badge, I was able to implement a protocol for the breeding of lighting patterns via sex.

![](http://bunniefoo.com/orchard/bm2015/bm2015-0th-order.gif)

![](http://bunniefoo.com/orchard/bm2015/bm2015-1st-order.gif)

![](http://bunniefoo.com/orchard/bm2015/bm2015-2nd-order.gif)

![](http://bunniefoo.com/orchard/bm2015/bm2015-fasthue.gif)

*Some examples of the unique light patterns possible through sex.*

**Sex**

When most people think of sex, what they are actually thinking about is sexual intercourse. This is understandable, as technology allows us to have lots of sexual intercourse without actually accomplishing sexual reproduction. Still, the double-entendre of saying “Nice lights! Care to have sex?” is a playful ice breaker for new interactions between camp mates.

Sex, in this case, is used to breed the characteristics of the badge’s light pattern as defined through a virtual genome. Things like the color range, blinking rate, and saturation of the light pattern are mapped into a set of diploid (two copies of each gene) chromosomes ( [code](https://github.com/bunnie/chibios-orchard/blob/bm15/orchard/genes.h)) ( [spec](https://github.com/bunnie/chibios-orchard/blob/bm15/orchard/genes.md)). Just as in biological sex, a badge randomly picks one copy of each gene and packages them into a sperm and an egg (every badge is a hermaphrodite, much like plants). A badge’s sperm is transmitted wirelessly to another host badge, where it’s mixed with the host’s egg and a new individual blending traits of both parents is born. The new LED pattern replaces the current pattern on the egg donor’s badge.

Biological genetic traits are often analog, not digital – height or weight are not coded as discrete values in a genome. Instead, observed traits are the result of a complex blending process grounded in the minutiae of metabolic pathways and the efficacy of enzymes resulting from the DNA blueprint and environment. The manifestation of binary situations like recessive vs. dominant is often the result of a lot of gain being applied to an analog signal, thus causing the expressed trait to saturate quickly if it’s expressed at all.

In order to capture the wonderful diversity offered by sex, I implement quantitative traits in the light genome. Instead of having a single bit for each trait, it’s a byte, and there’s an [expression function](https://github.com/bunnie/chibios-orchard/blob/bm15/orchard/genes.c#L100) that combines the values from each gene (alleles) to derive a final observed trait (phenotype).

By carefully picking expression functions, I can control how the average population looks. Let’s consider saturation (I used an [HSV colorspace](https://en.wikipedia.org/wiki/HSL_and_HSV), instead of RGB, which makes it much easier to create aesthetically pleasing color combinations). A highly saturated color is vivid and bright. A less saturated color appears pastel, until finally it’s washed out and looks just white or gray (a condition analogous to albinism).

If I want albinism to be rare, and bright colors to be common, the expression function could be a saturating add. Thus, even if one allele (copy of the gene) has a low value, the other copy just needs to be a modest value to result in a bright, vivid coloration. Albinism only occurs when both copies have a fairly low value.

![](http://bunniefoo.com/orchard/bm2015/bm2015-sat-dominant.png)

*Population makeup when using saturating addition to combine the maternal and paternal saturation values. Albinism – a badge light pattern looking white or gray – happens only when both maternal and paternal values are small. ‘S’ means large saturation, and ‘s’ means little saturation. ‘SS’ and ‘Ss’ pairings of genes leads to saturated colors, while only the ‘ss’ combination leads to a net low saturation (albinism).*

On the other hand, if I wanted the average population to look pastel, I can simply take the average of each allele, and take that to be the saturation value. In this case, a bright color can only be achieved in both alleles have a high value. Likewise, an albino can only be achieved if both alleles have a low value.

![](http://bunniefoo.com/orchard/bm2015/bm2015-sat-average.png)

*Population makeup when using averaging to combine the maternal and paternal saturation values. The most common case is a pastel palette, with vivid colors and albinism both suppressed in the population.*

For Burning Man, I chose saturating addition as the expression function, to have the population lean toward vivid colors. I implemented other features such as cyclic dimming, hue rotation, and color range [using similar techniques](https://github.com/bunnie/chibios-orchard/blob/orchard-r2/orchard/genes.md).

It’s important when thinking about biological genes to remember that they aren’t like lines of computer code. Rather, they are like the knobs on an analog synth, and the resulting sound depends not just on the position of the knob, but where it is in the signal chain how it interacts with other effects.

**Gender and Consent**

Beyond genetics, there is a minefield of thorny decisions to be made when implementing the social policies and protocols around sex. What are the gender roles? And what about consent? This is where technology and society collide, making for a fascinating social experiment.

I wanted everyone to have an opportunity to play both gender roles, so I made the badges hermaphroditic, in the sense that everyone can give or receive genetic material. The “maternal” role receives sperm, combines it with an egg derived from the currently displayed light pattern, and replaces its light pattern with a new hybrid of both. The “paternal” role can transmit a sperm derived from the currently displayed pattern. Each badge has the requisite ports to play both roles, and thus everyone can play the role of male or female simply by being either the originator of or responder to a sex request.

This leads us to the question of consent. One fundamental flaw in the biological implementation of sex is the possibility of rape: operating the hardware doesn’t require mutual consent. I find the idea of rape disgusting, even if it’s virtual, so rape is disallowed in my implementation. In other words, it’s impossible for a paternal badge to force a sperm into a maternal badge: male roles are not allowed to have sex without first being asked by a female role. Instead, the person playing the female role must first initiate sex with a target mate. Conversely, female roles can’t steal sperm from male roles; sperm is only generated after explicit consent from the male. Assuming consent is given, a sperm is transmitted to the maternal badge and the protocol is complete. This two-way handshake assures mutual consent.

This non-intuitive and partially role-reversed implementation of sex lead to users asking support questions akin to “I’m trying to have sex, but why am I constantly being denied?” and my response was – well, did you ask your potential mate if it was okay to have sex first? Ah! [Consent](https://www.youtube.com/watch?v=oQbei5JGiT8). The very important but often overlooked step before sex. It’s a socially awkward question, but with some practice it really does become more natural and easy to ask.

Some users were enthusiastic early adopters of explicit consent, while others were less comfortable with the question. It was interesting to see the ways straight men would ask other straight men for sex – they would ask for “ahem, blinky sex” – and anecdotally women seemed more comfortable and natural asking to have sex (regardless of the gender of the target user).

As an additional social experiment, I introduced a “rare” trait ( [pegged at ~3%](https://github.com/bunnie/chibios-orchard/blob/bm15/orchard/led.c#L316) of a randomly generated population) consisting of a single bright white pixel that cycles around the LED ring. I wanted to see if campmates would take note and breed for the rare trait simply because it’s rare. At the end of the week, more people were expressing the rare phenotype than at the beginning, so presumably some selective breeding for the trait did happen.

In the end, I felt that having sex to breed interesting light patterns was a lot more fun for everyone than tweaking knobs and sliders in a UI. Also, because traits are inherited through sexual reproduction, by the end of the event one started to see families of badges gaining similar traits, but thanks to the randomness inherent in sex you could still tell individuals apart in the dark by their light patterns.

**Finding Friends**

Implementing sex requires a peer-to-peer radio. So why not also use the radio to help people locate nearby friends? Seems like a good idea on the outside, but the design of this system is a careful balance between creating a general awareness of friends in the area vs. creating a messaging client.

Personally, one of the big draws of going to Burning Man is the ability to unplug from the Internet and live in an environment of intimate immediacy – if you’re physically present, you get 100% of my attention; otherwise, all bets are off. Email, SMS, IRC, and other media for interaction (at least, I hear there are others, but I don’t use them…) are great for networking and facilitating business, but they detract from focusing on the here and now. For me there’s something ironic about seeing a couple in a fancy restaurant, both hopelessly lost staring deeply into their smartphones instead of each other’s eyes. Being able to set an auto-responder for two weeks which states that your email will never be read is pretty liberating, and allows me to open my mind up to trains of thought that can take days to complete. Thus, I really wanted to avoid turning the badge into a chat client, or any sort of communication medium that sets any expectation of reading messages and responding in a timely fashion.

On the other hand, meeting up with friends at Burning Man is terribly hard. It’s life before the cell phone – if you’re old enough to remember that. Without a cell phone, you have a choice between enjoying the music, stalking around the venue to find friends, or dancing in one spot all night long so you’re findable. Simply knowing if my friends have finally showed up is a big help; if they haven’t arrived yet, I can get lost in the music and check out the sound in various parts of the venue until they arrive.

Thus, I designed a [very simple protocol](https://github.com/bunnie/chibios-orchard/blob/orchard-r2/orchard/orchard-app.c#L257) which will only reveal if your friends are nearby, and nothing else. Every badge emits a broadcast ping every couple of seconds. Ideally, I’d use an RSSI (receive signal strength indicator) to figure out how far the ping is, but due to a quirk of the radio hardware I was unable to get a reliable RSSI reading. Instead, every badge would listen for the pings, and [decrement the ping count](https://github.com/bunnie/chibios-orchard/blob/orchard-r2/orchard/orchard-app.c#L195) at a slightly slower average rate than the ping broadcast. Thus, badges solidly within radio range would run up a ping count, and as people got farther and farther away, the ping count would decrease as pings gradually get lost in the noise.

![](http://bunniefoo.com/orchard/bm2015/bm2015-friend-select.jpg)

*Friend finding UI in action. In this case, three other badges are nearby, SpacyRedPhage, hap, and happybunnie:-). SpacyRedPhage is well within range of the radio, and the other two are farther away.*

The system worked surprisingly well. The reliable range of the radio worked out to be about 200m in practice, which is about the sound field of a major venue at Burning Man. It was very handy for figuring out if my friends had left already for the night, or if they were still prepping at camp; and there was one memorable reunion at sunrise where a group of my camp mates drove our beloved art car, [Dr. Brainlove](https://www.facebook.com/drbrainlove), to [Robot Heart](https://www.facebook.com/RobotHeart) and I was able to quickly find them thanks to my badge registering a massive amount of pings as they drove into range.

**Hardware Details**

I’m not so lucky that I get to design such a complex piece of hardware exclusively for a pursuit as whimsical as Burning Man. Rather, this badge is a proof-of concept of a larger effort to develop a [new open-source platform for networked embedded computers](http://www.kosagi.com/w/index.php?title=Orchard_Main_Page) (please don’t call it IoT) backed by a rapid deployment supply chain. Our codename for the platform is [Orchard](http://www.kosagi.com/w/index.php?title=Orchard_Main_Page).

The Burning Man badge was our first end-to-end test of Orchard’s “supply chain as a service” concept. The core reference platform is fairly well-documented [here](http://www.kosagi.com/w/index.php?title=Orchard_Main_Page), and as you can see looks nothing like the final badge.

[![](http://bunniefoo.com/orchard/bm2015/bm2015-design-contrast_sm.jpg)](http://bunniefoo.com/orchard/bm2015/bm2015-design-contrast.jpg)

*Bottom: orchard reference design; top: orchard variant as customized for Burning Man.*

However, the only difference at a schematic level between the reference platform and the badge is the addition of 14 extra RGB LEDs, the removal of the BLE radio, and redesign of the captouch electrode pattern. Because the BOM of the badge is a strict subset of the reference design, we were able to go from a couple prototypes in advance of a [private Crowd Supply campaign](https://www.crowdsupply.com/bunnie/the-institute-blinky-badge-bm2015-private-campaign) to 85 units delivered at the door of camp mates in about 2.5 months – and the latency of shipping units from China to front doors in the US accounts for one full month of that time.

[![](https://www.crowdsupply.com/img/ba30/the-institute-badge-touch-1.gif)](https://www.crowdsupply.com/bunnie/the-institute-blinky-badge-bm2015-private-campaign)

*The badge sports an interactive captouch surface, an OLED display, 900MHz ISM band peer-to-peer radio, microphone, accelerometer, and more!*

If you’re curious, you can view documentation about the Orchard platform [here](http://www.kosagi.com/w/index.php?title=Orchard_Main_Page), and discuss it at the [Kosagi forum](http://www.kosagi.com/forums/index.php).

**Reflection**

As an engineer, my “default” existence is confined on four sides by cost, schedule, quality, and specs, with a sprinkling of legal, tax, and regulatory constraints on top. It’s pretty easy to lose your creative spark when every day is spent threading the needle of profit and loss.

Even though the implementation of Burning Man’s principles of decommodification and gifting is [far from perfect](http://www.washingtonpost.com/news/morning-mix/wp/2015/09/13/this-quiznos-spoof-ad-is-so-real-the-burning-man-festival-is-threatening-to-sue/), it’s sufficient to enable me to loosen the shackles of my daily existence and play with technology as a medium for enhancing human interactions, and not simply as a means for profit. In other words, thanks to the values of the community, I’m empowered and supported to build stuff that wouldn’t make sense for corporate shareholders, but might improve the experiences of [my closest friends](http://thephage.org/). I think this ability to leave daily existence behind for a couple weeks is important for staying balanced and maintaining perspective, because at least for me maximizing profit is rarely the same as maximizing happiness. After all, a warm smile and a heartfelt hug is priceless.
