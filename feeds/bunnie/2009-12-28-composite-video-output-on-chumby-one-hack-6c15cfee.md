---
title: Composite Video Output on Chumby One (Hack)
url: https://www.bunniestudios.com/blog/2009/composite-video-output-on-chumby-one-hack/
published: "2009-12-28T09:46:03Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=792
---

# Composite Video Output on Chumby One (Hack)

[![](http://bunniestudios.com/blog/images/c1video_twitter_sm.jpg)](http://bunniestudios.com/blog/images/c1video_twitter.jpg)

xobs recently completed a hack that adds a composite video output to the Chumby One. You can [read step by step details on how to do it](http://wiki.chumby.com/mediawiki/index.php/Add_a_video_connector_to_the_chumby_One) on the chumy wiki. The hack takes advantage of the fact that the CPU, a Freescale i.MX233, has a built-in composite video output that’s not connectorized, but available on the motherboard: you can solder a video cable to a set of pads as shown below (click for a larger version).

[![](http://bunniestudios.com/blog/images/c1video_inside_sm.jpg)](http://bunniestudios.com/blog/images/c1video_inside.jpg)

With a little case modding, you can have a very tidy video connector on the back as well.

![](http://bunniestudios.com/blog/images/c1video_backside.jpg)

In addition to the hardware mods, you need to install a modified kernel that contains the drivers for the video output, and call some scripts to switch the LCD output to the video output ( [instructions here](http://wiki.chumby.com/mediawiki/index.php/Add_a_video_connector_to_the_chumby_One)). Unfortunately, you can only drive one screen at a time with the i.MX233, so it’s not the easiest to interact with, but it is pretty neat for passively displaying information on your TV. Just configure the chumby with your favorite channel, plug it into an unused composite input, and watch photo slideshows from your favorite photosharing website, news headlines, Twitter feeds, eBay auction status, etc. Or, you can use it to make your TV chuck-tastic:

![](http://bunniestudios.com/blog/images/c1video_chucknorris.jpg)
