---
title: Web browser on the Infocast
url: https://www.bunniestudios.com/blog/2010/web-browser-on-the-infocast/
published: "2010-09-04T07:52:11Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1294
---

# Web browser on the Infocast

A couple months back, [Name that Ware](http://www.bunniestudios.com/blog/?p=1140) featured the [Insignia Infocast](http://www.bestbuy.com/site/Insignia+-+Infocast+8%22+Internet+Media+Display/9854795.p?id=1218185322584&skuId=9854795&st=infocast&cp=1&lp=1) by [Best Buy Exclusive Brands](http://en.wikipedia.org/wiki/Best_Buy#Best_Buy_exclusive_brands). While it’s marketed as a device for viewing [chumby](http://www.chumby.com/) apps and sharing photos, as far as the DIY crowd is concerned, the Infocast is a $169, 800 MHz linux machine with an SVGA touchsreen, 128 MB of DDR2, and a 2GB disk drive.

[![](http://bunniestudios.com/blog/images/ChumbyWithKeyboard2_sm.jpg)](http://bunniestudios.com/blog/images/ChumbyWithKeyboard2.jpg)

An example of the versatility of the platform is [hb](http://ken.ringzero.net/)‘s recent port of the [Qt](http://qt.nokia.com/products/) UI framework running [webkit](http://webkit.org/) to the [Infocast](http://www.bestbuy.com/site/Insignia+-+Infocast+8%22+Internet+Media+Display/9854795.p?id=1218185322584&skuId=9854795&st=infocast&cp=1&lp=1), pictured running above. For those who want to build it themselves, there are [instructions on the chumby wiki](http://wiki.chumby.com/mediawiki/index.php/Web_Browser) and a [forum for questions](http://forum.chumby.com/viewtopic.php?pid=31026); or you can just download a pre-packaged [binary image](http://files.chumby.com/browser/chumby_silvermoon_browser-1.0.zip) that you can uncompress to a USB thumb drive, toss it into one of the ports on the back, reboot and use. Note that the implementation assumes a USB keyboard plugged for text input.

Of course, this is just scratching the surface on what you can do with the platform. There are folks working on porting Android and OpenEmbedded, and the [hardware reference schematics](http://files.chumby.com/bunnie/silvermoon_oem/silvermoon_OEM_ref_v3.pdf) are available for those inclined to the soldering iron.
