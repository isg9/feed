---
title: Fabrication and Characterization of N-Channel Enhancement Mode Insulated Gate Field Effect Transistors
url: https://sam.zeloof.xyz/fet/
published: "2016-12-30T15:26:02Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=182
---

# Fabrication and Characterization of N-Channel Enhancement Mode Insulated Gate Field Effect Transistors

Update 4/25/18: [First IC!](http://sam.zeloof.xyz/first-ic/)

Update 8/27/17: [Developing Patterning Process for Homemade Microelectronics](http://sam.zeloof.xyz/developing-patterning-process-for-homemade-microelectronics/)

Update 7/30/17: [Developing Metalization Process for Homemade Microelectronics](http://sam.zeloof.xyz/developing-metalization-process-for-homemade-microelectronics/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/nmos-fet-fabrication-300x225.jpg)](https://sam.zeloof.xyz/fet/nmos-fet-fabrication/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/Nmos-fet-construction-300x225.jpg)](https://sam.zeloof.xyz/fet/nmos-fet-construction/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6867-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6867/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6866-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6866/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_9239-300x225.jpg)](https://sam.zeloof.xyz/fet/img_9239/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6900-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6900/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6896-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6896/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6901-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6901/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/D7V6902-1-300x199.jpg)](https://sam.zeloof.xyz/fet/_d7v6902-2/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_9238-e1488587620760-300x199.jpg)](https://sam.zeloof.xyz/fet/img_9238/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_9233-300x225.jpg)](https://sam.zeloof.xyz/fet/img_9233/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_9232-300x225.jpg)](https://sam.zeloof.xyz/fet/img_9232/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_9231-300x225.jpg)](https://sam.zeloof.xyz/fet/img_9231/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/D7V7331-e1524952531619-300x225.jpg)](https://sam.zeloof.xyz/developing-patterning-process-for-homemade-microelectronics/_d7v7331/)

Particles shorting gate

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/active-1-300x225.png)](https://sam.zeloof.xyz/active-1/)

Active area

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/gate-2-300x225.png)](https://sam.zeloof.xyz/gate-2/)

Gate oxide

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/contact-3-300x225.png)](https://sam.zeloof.xyz/contact-3/)

Source/Drain contact

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/metal-4-300x225.png)](https://sam.zeloof.xyz/metal-4/)

Top metal

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/08/FET-final-300x225.png)](https://sam.zeloof.xyz/fet-final/)

Composite image

These are my first working transistors. Specifically, they are insulated gate enhancement mode n channel field effect transistors. I also made a depletion mode FET with a conducting channel and it worked even better than the enhancement mode ones. I drew out the steps I took to make this, they’re based on Jeri Ellsworth’s work but with a few main changes. It is very important that your dielectric overlap source and drain regions on the FET, otherwise no inversion layer can be formed and the FET cannot turn on. Only a small overlap is necessary and the larger it is, the more unwanted capacitance there is. This is not normally a consideration in actual production because inherent lateral diffusion takes care of the overlap but for these large hand made devices I found you have to get very lucky with your alignment if you don’t budget extra overlap.

A huge thanks to Jeri for making her videos about home chip fabrication which got me interested in these experiments in the first place.

You can see in the center of the transistor there is a red region of silicon dioxide, this is the gate and the color indicates that it is roughly 750 angstroms thick. I would like to make the gate thinner so I can achieve lower threshold voltages, etc. but it is hard to make a truly insulating gate much thinner than that in a dirty environment because of pinholes and other impurities in the oxide layer.

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2657-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2657.jpg)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2659-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2659.jpg)

[![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2661-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2016/12/IMG_2661.jpg)

Diode characteristics are characterized with a semiconductor parameter analyzer

A brief introduction to semiconductor fabrication processes and terminology. It is not intended to be an in depth view of any single process, but rather an overview so that provides enough information for someone to get started with making diodes and transistors at home.

Tour of my home chip fab setup in early 2017. I’ve been accumulating this equipment since October of 2016.

Step by step FET fabrication
