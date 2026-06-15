---
title: Bitbake, Cake, and Black Friday
url: https://www.bunniestudios.com/blog/2010/bitbake-cake-and-black-friday/
published: "2010-11-23T16:27:24Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1435
---

# Bitbake, Cake, and Black Friday

So, first the pitch — for your Black Friday shopping pleasure, here’s a couple of things that will help scratch that itch to play around with hardware.

First of all, two chumby-powered devices under the Insignia brand in Best Buy are on sale for Black Friday. The [original 8″ Infocast](http://www.bunniestudios.com/blog/?p=1140) device — an [800 MHz linux PC](http://www.bunniestudios.com/blog/?p=1294) with an 8″ SVGA LCD and touchscreen — is rumored to be on sale for [as low as $99](http://www.theblackfriday.com/ads/bbbf/bestbuy-black-friday-ad3.shtml) in some stores, although Best Buy’s [on-line price](http://www.bestbuy.com/site/Insignia%26%23153%3B+-+Infocast+8%22+Internet+Media+Display/9854795.p?id=1218185322584&skuId=9854795&st=infocast&cp=1&lp=1) pegs it at $129. Either way, it’s a smashing deal for a linux PC.

[![](http://bunniestudios.com/blog/images/infocast_99bucks.jpg)](http://www.theblackfriday.com/ads/bbbf/bestbuy-black-friday-ad3.shtml)

The other chumby-powered device is the 3.5″ Infocast, which you can think of as the [Chumby One Internet Radio’s](http://www.amazon.com/gp/product/B0030QUU4M?ie=UTF8&tag=bunniestudios-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=B0030QUU4M)![](http://www.assoc-amazon.com/e/ir?t=bunniestudios-20&l=as2&o=1&a=B0030QUU4M) battery-less little brother. This, too, is on sale for Black Friday, and at [just $79](http://www.bestbuy.com/site/Insignia%26%23153%3B+-+Infocast+3.5%22+Internet+Media+Display/1152881.p?id=1218226456157&skuId=1152881&st=infocast&cp=1&lp=2), it’s a steal.

[![](http://images.bestbuy.com/BestBuy_US/images/products/1152/1152881_rb.jpg)](http://www.bestbuy.com/site/Insignia%26%23153%3B+-+Infocast+3.5%22+Internet+Media+Display/1152881.p?id=1218226456157&skuId=1152881&st=infocast&cp=1&lp=2)

The motherboard for the 3.5″ Infocast is shown below, and [if it looks familiar](http://www.bunniestudios.com/blog/?p=611), you’d be right.

[![](http://bunniestudios.com/blog/images/infocast_35_front_sm.jpg)](http://bunniestudios.com/blog/images/infocast_35_front.jpg)

[![](http://bunniestudios.com/blog/images/infocast_35_back_sm.jpg)](http://bunniestudios.com/blog/images/infocast_35_back.jpg)

One small improvement to the 3.5″ Infocast board is the addition of a “mod port” which breaks out several GPIOs, I2C, composite video, and a spare USB port to a 0.1″ pitch header (which is a subset of the pinout that’s featured in the [chumby hacker board](http://www.adafruit.com/index.php?main_page=product_info&products_id=278)), shown below.

![](http://bunniestudios.com/blog/images/infocast_35_modport.jpg)

**sudo find me a Cake!**

So here’s where the Cake comes in. Somewhere on the motherboard of the Infocast 3.5″, I’ve hidden a cake — the [cake is *not* a lie](http://www.youtube.com/watch?v=qdrs3gr_GAs). The first person who can find the cake and post a photo of it within the next two months will be given a cash prize of US$300. Put in perspective, that’s two months’ wage for a Chinese factory worker (and that’s *after* the across-the-industry 20% raises that were effected by a [string of suicides at Foxconn](http://www.guardian.co.uk/world/2010/may/28/foxconn-plant-china-deaths-suicides), where the iPhone is assembled). Given that the standard monthly prize for Name that Ware is just $10, this gives an idea of how hard I think it is to find the cake. But, I promise you — the cake is not a lie.

**Bitbake it yourself!**

And here’s where the [Bitbake](http://en.wikipedia.org/wiki/BitBake) comes in. For those who have been wanting to build your own firmware for both the 8″ and 3.5″ chumby platforms, a simple solution finally exists. As [announced on the chumby forum](http://forum.chumby.com/viewtopic.php?pid=32803), there is now an [easy to use Open Embedded](http://wiki.chumby.com/mediawiki/index.php/Building_OpenEmbedded_%28Beta%29#Building_with_OpenEmbedded) configuration that will allow you to build and customize, from scratch, your very own firmware image.

You can configure the firmware to be anything from a minimal console-only build, to one that includes a window system and web browser of your choosing. The build configuration is “complete” in the sense that the product of the [OpenEmbedded](http://wiki.openembedded.org/index.php/Main_Page) build is a binary image that you can directly write to the microSD card and boot with no need for further massaging.

Have a happy holiday weekend!
