---
title: Make Your Own 3G Router
url: https://www.bunniestudios.com/blog/2010/make-your-own-3g-router/
published: "2010-04-29T05:33:02Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1076
---

# Make Your Own 3G Router

There’s an Easter Egg inside the [chumby One](http://www.costco.com/Browse/Product.aspx?Prodid=11529965&search=chumby) (which is now on sale at [Costco online for $99](http://www.costco.com/Browse/Product.aspx?Prodid=11529965&search=chumby), cheaper than chumby.com’s $119 price!) that enables it to work with certain 3G USB modems and function as a 3G router (similar to a [MiFi](http://www.verizonwireless.com/b2c/mobilebroadband/?page=products_mifi)).

![](http://bunniestudios.com/blog/images/verizon_3g_c1.jpg)

Above is a picture of a [Verizon USB760 modem](http://reviews.cnet.com/modems/verizon-wireless-usb760-modem/4505-3004_7-33411169.html) plugged into the USB port of a chumby One.

Use of the Easter Egg is documented on the [chumby wiki](http://wiki.chumby.com/mediawiki/index.php/Connect_through_3G), but it can be quite simple:

> 1\. Plug in your 3G card.
>
> 2\. Go to the Network Config screen by pressing the top button, going to Settings, then tapping on Network.
>
> 3\. Select “create a new connection”, press OK, and wait for the scan to time out.
>
> 4\. Select “Other”
>
> 5\. Enter 3G as the name of the access point. It must have a capital G for it to work.
>
> 6\. Wait for the subsequent scan to time out.
>
> 7\. Select “Manual”.
>
> 8\. Tap “OK” to select None as the encryption type.
>
> 9\. Tap “OK” to select Automatic IP allocation.
>
> 10\. Tap “OK” to confirm settings and connect to the network

To activate local wifi sharing, ssh into the device (or plug in a USB keyboard to activate the drop-down console) and use the command “touch /psp/start\_ap” to flag the network setup scripts to configure your chumby One as an access point. Note that AP mode also works with other network sources, such as Ethernet-over-USB.

![](http://bunniestudios.com/blog/images/verizon_3g_c1_cnxn.jpg)

This works because the chumby One ships with a copy of pppd, hostapd and a set of glue scripts (courtesy of xobs) that allow it first to talk to the 3G network, and second to work as an access point that shares the 3G connection with other wifi devices using its internal wifi card.

![](http://bunniestudios.com/blog/images/3g_schema.jpg)

Since this is an Easter Egg, it has a few rough edges on it, such as no native UI to do things like configure your AP SSID and encryption mode, and for many GSM networks you’ll need to enter a custom username, password and phone number; but you certainly can [ssh in](http://wiki.chumby.com/mediawiki/index.php/Chumby_tricks#Open_a_secure_shell_console_on_the_chumby) and tweak the scripts to set those things up. Regardless, this is a very handy trick that I’ve used in hotels from Shenzhen to Tokyo to New York to set up a wifi hotspot (complete with Internet radio and chumby apps) on the road — the hack works with quite a few 3G dongles and networks (given a little tweaking).
