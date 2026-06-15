---
title: Chumby Wifi Sniffer
url: https://www.bunniestudios.com/blog/2008/chumby-wifi-sniffer/
published: "2008-04-07T12:45:09Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=234
---

# Chumby Wifi Sniffer

For [ETech08](http://en.oreilly.com/et2008/public/content/home), I built the “Chumby Tower” and one of the hacks featured in the tower is a version of the chumby that sniffs wifi and renders captured packets onto the display. The chumby is a great device for passively monitoring wifi signals, because it has a screen for rendering the data real-time, sufficient CPU power to do the data processing, and it’s cheap enough that you don’t mind dedicating it to that application. It’s handy, for example, to see if your neighbors are stealing your wifi link to surf the web; at a glance you can see what URLs are going by over the air.

![](http://bunniestudios.com/blog/images/tshark_screencap.jpg)

*Someone has their RSS feed reader set to [wowinsider.com](http://www.wowinsider.com/)… ;-)*

The sniffer application sets the wifi radio in the chumby to monitor mode, so it picks up any and all packets on the selected channel. The application then relies on a cross-compiled version of tshark and dumpcap to capture 802.11 packets in raw form. The dumps are processed with a perl script that identifies certain headers and keywords that delimit strings of interest. These strings are then output in an XML container, which is then read by a Flash movie and rendered to the screen. You can adjust what interesting strings are selected by modifying the perl script. Currently, the script just extracts http link strings, To/From/Subject of unencrypted POP mail, and HTML page titles and headers. However, it’s fairly easy to extend the script to do things like capture plaintext username/password pairs that are transmitted over wifi, which happens surprisingly often.

For those who just want to use the sniffer, you can download [this zipfile](http://files.chumby.com/bunnie/sniffer.zip) and extract it to a blank USB drive, plug it into a chumby, reboot, and when the chumby comes back it should be running the application. At the point where the screen goes dark, it will take up to a minute for data to start appearing, as it takes about 20-30 seconds for the huge tshark executable to load (depending upon the performance of your USB drive), and then the script pools data over a few sampling periods before reporting summary information. For those who wish to hack the program a bit, here is a link to the [Flash movie](http://bunniestudios.com/blog/images/floatText.fla)—it’s the only piece of code in the zipfile that isn’t available in source form some other way.

The default frequency of the radio will be the last frequency used, but you can add a line to the “userhook2” perl script right after monitor mode is set to configure the channel explicitly:

```
iwconfig rausb0 channel 3   # set channel number to desired number

```

The script is a little bit fragile, in that every time the firmware is updated something seems to break. I just repaired the script to work with firmware version 1.5, but let me know if you use the script and it doesn’t seem to work. Unfortunately, the only way to debug the script is via the serial port, because setting the wifi card in monitor mode makes it unable to transmit packets — although if you’re really clever you can hack the script to load drivers for a wired ethernet USB dongle and connect to the chumby via that interface.

For what it’s worth, cross-compiling tshark is a total pain in the butt. The make scripts really weren’t very friendly to the task, I had to do a lot of manual operations to facilitate the build. For example, it doesn’t differentiate between binaries that run on the host versus binaries that run on the target, so some helper programs meant to run on the x86 host would get complied to ARM (doh!); those cases had to be tweaked manually to get the whole thing to build right.
