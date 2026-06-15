---
title: Quake on Chumby
url: https://www.bunniestudios.com/blog/2008/quake-on-chumby/
published: "2008-08-29T09:41:06Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=262
---

# Quake on Chumby

xobs, a developer at chumby, showed me one of the coolest things I’ve seen in a while on a chumby: a full port of Quake. He got the whole thing running under SDL and hacked up the event layer so that the accelerometer (tilting) is used to move in the game, the bend switch is used to fire, and a touch anywhere on the screen is used to jump/activate items. So now you can hug a Linux computer *and* frag bad guys at the same time. Practical? no. Cool? yes.

![](http://bunniestudios.com/blog/images/quake_chumby.jpg)

Below is a video of the loading screen plus a quick sample of the gameplay.

\[flashvideo filename=http://files.chumby.com/hacks/quake\_gameplay.flv /\]

As you can see the frame rate is *very* playable on this little 350 MHz ARM9 i.MX21 CPU, although I had a little trouble holding the camera and playing at the same time, while finding a good angle for filming. I never really equated chumby in my mind with a video-game capable system like this, but that’s the joy of open source — people will do things with the hardware that you never expected or had in mind when you designed it. I love it!

You can find out more about how he did it in the chumby [developer forum](http://forum.chumby.com/viewtopic.php?id=2955), and if you own a chumby you can [download a USB dongle image](http://files.chumby.com/hacks/chumby_quake.tar.gz) that lets you play it (and if you don’t own a chumby, [you can get one](http://store.chumby.com/)). And, xobs was fastidious about keeping compliant with the GPL and immediately posted the [source](http://files.chumby.com/hacks/chumby_quake_src.tar.gz) tarball as well for you to peruse.
