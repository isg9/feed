---
title: chumby hacker boards (now available in beta)
url: https://www.bunniestudios.com/blog/2010/chumby-hacker-boards-now-available-in-beta/
published: "2010-08-30T18:58:40Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1274
---

# chumby hacker boards (now available in beta)

chumby is now offering a “hacker” board, which is the guts of the [chumby One](http://www.amazon.com/gp/product/B0030QUU4M?ie=UTF8&tag=bunniestudios-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=B0030QUU4M)![](http://www.assoc-amazon.com/e/ir?t=bunniestudios-20&l=as2&o=1&a=B0030QUU4M), but modified to be more hacker-friendly: it comes with three high speed USB host ports, uses the power connector from the Sony PSP (instead of the weird, hard to find connector on the chumby One) and incorporates a variety of headers, such as Arduino-style shield headers and a 44-pin breakout header that gives you access to a lot of digital I/O and some analog inputs. There’s even a four-directional switch on board and some LEDs so you can do quick hacks that don’t require a video display for user feedback. Speaking of the display, while this board doesn’t come standard with an LCD, it does provide composite video output via a 4-wire 1/8″ jack so you can, by using an iPod video cable, plug the chumby hacker board into any TV that supports a composite video input.

[![](http://wiki.ladyada.net/_media/chumbyhandbig.jpg?w=500)](http://www.adafruit.com/index.php?main_page=index&cPath=46)

(Photo by [Adafruit](http://adafruit.com))

The hacker board is currently being sold through [Adafruit](http://www.adafruit.com/index.php?main_page=index&cPath=46) and also through [Sparkfun](http://www.sparkfun.com/commerce/product_info.php?products_id=10106) as part of a limited-run beta program. The board is priced at around $89. The goal of the beta program is to collect feedback from users who purchase the board to fine-tune the design and to figure out what I/Os and accessories make sense to bundle with the board. Like the Arduino, we don’t integrate a lot of features onto the mainboard itself (keeps base cost low). Instead, we’d like to make sure that adequate I/O resources exist for developers to hack in the peripheral module they require to complete their project — or for more enterprising developers to build their own flavor of peripheral board and sell their own accessory.

There’s a few resources available to get people started on using the boards: a [forum](http://forum.chumby.com/viewforum.php?id=20) for general support and questions, and a [wiki](http://wiki.chumby.com/mediawiki/index.php/Chumby_hacker_board) containing links to datasheets, schematics, and other more permanent documentation that people will find useful. Adafruit also has available a snazzy [hackerboard page](http://wiki.ladyada.net/chumbyhackerboard) with tons of info, well-documented tutorials, and nice photos to boot.

One other point of note about the hacker board is that you can install a native gcc toolchain on it, so you don’t need to configure/install a cross-compiler on your host PC to develop for it. Heck, it’s got a 454 MHz CPU and plenty of disk space, so why not? Adafruit has [a tutorial on how to install the compiler](http://wiki.ladyada.net/chumbyhackerboard/compiler) using a downloadable self-extracting script and a USB dongle. I’ve also heard rumors that an [OpenEmbedded](http://wiki.openembedded.net/index.php/Main_Page) port is coming to the board soon, so stay tuned.

If you do end up purchasing a board and participating in the beta, please do contribute to the fora and wikis with your feedback. As always, happy hacking!
