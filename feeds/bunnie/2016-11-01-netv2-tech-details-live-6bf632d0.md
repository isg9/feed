---
title: NeTV2 Tech Details Live
url: https://www.bunniestudios.com/blog/2016/netv2-details-live/
published: "2016-11-01T09:35:00Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=4842
---

# NeTV2 Tech Details Live

[![](http://bunniefoo.com/netv2/netv2-front_thumb.jpg)](http://bunniefoo.com/netv2/netv2-front.jpg)[![](http://bunniefoo.com/netv2/netv2-back_thumb.jpg)](http://bunniefoo.com/netv2/netv2-back.jpg)

Alphamax LLC now has [details of the NeTV2 live](https://alphamaxmedia.com/w/index.php?title=NeTV2), including links to preliminary schematics and PCB source files.

The key features of NeTV2 include:

- mPCIE v2.0 (5Gbps x1 lane) add-in card format

- Support for full 1080p60 video
- Artix-7 FPGA
- FPGA “hack port” breaking out 3x spare GTP transceiver pairs
- 512 MB of DDR3-800 @ 32-bit wide memory for frame buffering

I adopted an add-in card format to allow end users to pick the cost/performance trade-off that suited their application the best. Some users require only a text overlay (NeTV’s original design scenario); but others wanted to blend HD video and 3D graphics, which would require a substantially more powerful and expensive CPU. An add-in card allows users to plug into anything from an [economical $60 all-in-one](https://www.amazon.com/gp/product/B00G237LLA/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B00G237LLA&linkCode=as2&tag=bunniestudios-20&linkId=a9c8337132318567f64ff2b506601bb4)![](//ir-na.amazon-adsystem.com/e/ir?t=bunniestudios-20&l=am2&o=1&a=B00G237LLA), to a fully loaded gaming machine. The [kosagi forum](https://www.kosagi.com/forums/viewtopic.php?pid=2964#p2964) has an open thread for NeTV2 discussion.

[As noted previously](http://www.bunniestudios.com/blog/?p=4782), we are [currently seeking legal clarity](https://www.eff.org/press/releases/eff-lawsuit-takes-dmca-section-1201-research-and-technology-restrictions-violate) on the suite of planned features for the product, including highly requested features such as alpha blending which require access to the descrambled video stream.
