---
title: Developing Apps for Your TV the Easy and Open Way
url: https://www.bunniestudios.com/blog/2018/developing-apps-for-your-tv-the-easy-and-open-way/
published: "2018-06-05T16:01:58Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5331
---

# Developing Apps for Your TV the Easy and Open Way

The biggest screen in your house would seem a logical place to integrate cloud apps, but TVs are walled gardens. While it’s easy enough to hook up a laptop or PC and pop open a browser, there’s no simple, open framework for integrating all that wonderful data over

the TV’s other inputs.

[![](https://bunniefoo.com/netv2/netv2-cx-infographic.png)](https://bunniefoo.com/netv2/netv2-cx-infographic.png)

Until now. Out of the box, NeTV2’s “NeTV Classic Mode” makes short work of overlaying graphics on top of any video feed. And thanks to the Raspberry Pi bundled in the Quickstart version, NeTV2 app developers get to choose from a diverse and well-supported ecosystem of app frameworks to install over the base Raspbian image shipped with every device.

For example, Alasdair Allan’s [article on using the Raspberry Pi with Magic Mirror and Google AIY](https://medium.com/@aallan/a-magic-mirror-powered-by-aiy-projects-and-the-raspberry-pi-e6a0fea3b4d6) contains everything you need to get started on turning your TV into a voice-activated personal assistant. I gave it a whirl, and in just one evening I was able to concoct the demo featured in the video below.

Magic Mirror is a great match for NeTV2, because all the widgets are formatted to run on a black background. Once loaded, I just had to set the NeTV2’s chroma key color to black and the compositing works perfectly. Also, Google AIY’s Voicekit framework “just worked” out of the box. The only fussy bit was configuring it to work with my USB microphone, but thankfully there’s a [nice Hackaday article detailing exactly how to do that](https://hackaday.com/2017/05/30/diy-google-aiy/).

Personally, I find listening to long-form replies from digital assistants like Alexa or Google Home a bit time consuming. As you can see from this demo, NeTV2 lets you build a digital assistant that pops up data and notifications over the biggest screen in your house using rich visual formats. And the best part is, when you want privacy, you can just unplug the microphone.

If you can develop an app that runs on a Raspberry Pi, you already know everything you need to integrate apps into any TV. Thanks to NeTV2, there’s never been an easier or more open way to make the biggest screen in your house the smartest screen.

The NeTV2 is [crowdfunding now at CrowdSupply.com](https://crowdsupply.com/alphamax/netv2), and we’re just shy of our first stretch goal: a *free* Tomu bundled with every board. Normally priced at $30, [Tomu](https://tomu.im) is a tiny open-source computer that fits in a USB Type-A port, and it’s the easiest way to add an extra pair of status LEDs to an NeTV2. Help unlock this deal by [backing now](https://crowdsupply.com/alphamax/netv2) and spreading the word!

[![](https://bunniefoo.com/netv2/netv2-slogan-white.png)](https://crowdsupply.com/alphamax/netv2)
