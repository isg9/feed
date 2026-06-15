---
title: DIY Doppler Radar &#8211; LNB Hack
url: https://sam.zeloof.xyz/diy-doppler-radar-lnb-hack/
published: "2016-12-30T03:13:53Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=59
---

# DIY Doppler Radar &#8211; LNB Hack

I modified a satellite LNB to transmit as well as receive on 10ghz and connected the downconverted and mixed signal to my scope. I can use it to measure doppler shift and therefore speed of a moving object. I did the modifications by removing HEMT RF FETs and rotating them so their gate is where their drain was and vice versa. The source connections stayed grounded. I then had to cut some of the traces on the board and change a few resistors and things to make the biasing circuity work in reverse. I then capacitively coupled the dielectric resonant local oscillator the the transmitting section that I made. Watch Jeri Ellsworth’s Videos: [https://www.youtube.com/watch?v=vDyo\_OQFdAc](https://www.youtube.com/watch?v=vDyo_OQFdAc)

Received signal is mixed with the local oscillator (10 ish ghz) such that the result is the subtraction of the two signals, aka the doppler shift which can be related to speed.

A similar mod can be done to use these as 10ghz ham radio transmitters: [http://www.qsl.net/pa3gco/zelfbouw/blauwkap/bluecap.html](http://www.qsl.net/pa3gco/zelfbouw/blauwkap/bluecap.html)

![](http://sam.zeloof.xyz/wp-content/uploads/2017/01/IMG_8212.jpg)
