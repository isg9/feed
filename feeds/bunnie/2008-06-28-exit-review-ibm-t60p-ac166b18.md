---
title: 'Exit Review: IBM T60p'
url: https://www.bunniestudios.com/blog/2008/exit-review-ibm-t60p/
published: "2008-06-28T10:20:29Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=256
---

# Exit Review: IBM T60p

Keeping in line with the [exit review](http://www.bunniestudios.com/blog/?p=242) series, I’ve written an exit review of my [IBM T60p](http://www.bunniestudios.com/blog/?p=146) laptop, which was recently retired.

The T60p was purchased back in March 2006, right when the first dual-core mobile chips started shipping. The IBM model number was 2623-DDU, and it featured an Intel T2500 (2GHz), 2GB RAM, 100GB 7200rpm HD, 15″ 1600×1200 LCD, 512MB ATI FireGL V5200, CDRW/DVDRW, Intel 802.11abg wireless, WWAN, Bluetooth/Modem, 1Gb Ethernet, UltraNav, Secure chip, Fingerprint reader, 9 cell Li-Ion batt, and WinXP Pro. Like most of my key gadgets, the T60p followed me everywhere around the world; I spend probably 16 hours a day with it, so it got plenty of wear and tear.

I retired my T60p because Microsoft is retiring Windows XP, and I really don’t want to downgrade to Vista (“congratulations, your new faster hardware now runs slower than ever thanks to Vista!”), so I decided to get the latest, greatest Windows XP laptop available today, and hunker down and hope it lasts me until Microsoft figures out how to get Vista right (or they realize what a terrible mistake they’ve made and bring back XP…sort of like how Intel brought out the Pentium 4 but now all the new designs are Pentium III derivatives). Or maybe before that, Linux support for all of my key apps will get so good, I can finally depart from the Windows ecosystem.

So the reason for getting rid of the T60p isn’t because it had any functional flaw; it’s actually in fair working order, and its performance is still decent.

Here were some of my favorite features about the T60p:

Integrated Verizon EVDO. This was a life-changer; the availability of broadband wireless anywhere in the US in an integrated fashion has literally changed the way I work and live, and it has greatly increased my productivity, especially when traveling. No more searching for free hotspots! Less worry about people sniffing my wireless signal!

1600 x 1200 screen. Can’t live without the pixels!

Good horsepower. Plays WoW well, although it struggled a bit with TBC expansion pack. Runs all my design software adequately.

Decent battery life. On a 9-cell battery, I get about 3.5 hours a charge. The 6-cell battery gave me about 2 hours, which is long enough for general use; typically I go around with the 6-cell battery as it’s lighter and easier to carry.

Nice keyboard. I’ve always liked the Thinkpad keyboards.

Fingerprint sensor. There’s something sexy about stroking your laptop to log into it, even if it is a security gimmick.

TrackPoint mouse. The TrackPoint is essential for precision CAD work, and one of the reasons I can’t use a whole class of laptops that lack this feature, including Apple laptops.

Nice looking. I like the austere black slab look of the Thinkpads — the Dell and HP laptops look so cheesy and plasticky, I feel like I’m carrying around a Fisher-Price toy when I have one of those. I hate to say it, but the Apple notebooks are also getting a bit trite, and from what I’ve seen metal tends to dent, deform and scratch, whereas plastic is more resilient.

Good accessories. The AC/DC adapter was the best — compatible with every voltage in the world, auto DC adapters *and* those funky EmPower adapters found on some airplanes. Also there was a pretty good parallel/serial adapter provided by IBM (as well as one integrated into the minidock), which is key for embedded hardware development.

Great thermal management system. The laptop’s fan is whisper quiet, but it pumps out a lot of heat. It’s great to be able to hold your hand about five inches away and feel the heat being quietly pumped out of the laptop — a true sign of good thermal design.

Great customer support. Every one of my interactions with warranty service was positive, with the exception of talking to the guy to renew my warranty service. He was pretty clueless, but it was a moot point since I got a new laptop anyways.

Solid construction. With the exception of a service-induced crack (more on that later), the T60p is in good condition. The clutch on the screen is still solid, and the hard drive is still in good condition (although it does operate noticeably slower, probably due to the bearings being degraded by years of operation in tough conditions, such as airplanes in turbulence and bumpy car rides in China).

Good uptime. My average uptime for my laptop was about two weeks without a reboot — this is through multiple sleep cycles and reconfigurations to my external dual-monitor rig, etc. The chief cause for reboots was actually windows running out of GDI resources due to application leaks.

Reasonably light and thin. I wish it were like the ultra-thin laptops, but hey, for the amount of performance I demand and the pixels I require on the LCD, this is probably the best compromise. I do keep a spare ultra-light laptop around for the times when I go on vacations to far-flung regions of the world and I can’t afford the weight of my primary laptop.

However, I also had many, many problems with the T60p as well. Here are some of the more significant ones:

About a year into using the laptop, the ThinkVantage access software blew up on me, rendering my internal WiFi card useless. This was triggered when I once installed an external Wifi card for testing purposes, and the installed drivers broke some internal configuration. I tried just about everything short of wiping the OS to get my WiFi card working again, but alas, I never got it back. It’s tragic, as it’s like I had a hardware failure due to a software error. Fortunately, I have copious quantities of USB Wifi dongles laying around thanks to my work, so I just carried one of those around all the time.

The Verizon Access software is flaky. I had a little ritual for getting it to work — turn off the radio, start the app, let the app crash once, turn the radio on, start the app again, and then it works. If you started the app with the radio on it’d never figure out that there was signal. What a weird wart.

The case near the trackpad cracked. This was due actually to a flawed repair job by a technician that replaced my LCD. The technician didn’t mate one of the friction-lock connectors on the keyboard bezel exactly, which created a stress point and eventually caused the panel to crack.

Speaking of which, the LCD had to be replaced once. The LCD failed by having a set of horizontal lines across the bottom of the panel (I recognized that failure as stress-related failure of the COB drivers on the panel). The replacement LCD had a flaw in it too — there was a spot of blue pixels, about ten pixels on a side, that were always stuck-on, just faintly. It wasn’t annoying enough for another return but still, it’s unfortunate that the replacement LCD had a significant flaw as well.

One of the DRAM SO-DIMMs failed about 1.5 years into the laptop’s life. It was manifested by lots of bluescreens, and then I ran some DRAM test software and identified the problem. Spray-cooling the DRAM chips fixed the problem, so I installed some extra heat sinking in the laptop, which got around the problem for another month. Eventually, I had to call in for a replacement of the chips, which was laudably painless. The IBM service rep just sent out the replacement chips right away, no questions asked, and everything worked fine from then on (the laptop was still under its extended warranty at this time). However, it does highlight the fact that there is a thermal design flaw in the DRAM area: there is no heat piping or heat sinking around it. I think that’s because most configurations use less DRAM; I really cranked up the DRAM configuration, which meant I installed the most extreme, power-hungry devices in my laptop, pushing the thermal design margin right to the edge.

The trackpoint mouse was flakey sometimes; the drivers would sometimes not load properly or crash, which is annoying.

The ThinkVantage system update software, while a nice concept, is a bit annoying to use as it has to update itself typically before updating the system. As a general note, I don’t like vendor-specific extensions to the OS. They are never very well done, and often break: “Dear vendor, please stop trying to ‘add value’. You actually are destroying the value of your brand. Just make your good hardware and don’t get in my way, kthx bai.”

The FireGL V5200 graphics chipset, while very powerful, had some serious compatibility limitations. The OpenGL drivers never really worked right, and it was totally unsupported by some games, such as Second Life. Unfortunately, graphics drivers for laptops lag the main branch, since it seems every laptop requires a bit of tweaking to the drivers by the vendor.

The UltraBay lithium polymer battery that I purchased failed — twice. The failure was manifested in the battery going to 60% capacity within two months of use. The first time I called it into warranty service and got a replacement, and the second time it failed I just decided it was a design flaw and gave up on replacing it. The UltraBay lithium polymer is handy for situations where you’re on battery power and you want to swap out the main battery without going into hibernation. Fortunately, that situation was rare and the battery life of the main batteries was good enough, so it wasn’t really a problem in practice.

This is more my fault than anything else, but 100 GB was too small to live in. It didn’t help that I had multiple VMware images on my machine and my hardware design tools often chew up over 3GB per tool. The 2 GB of DRAM was also tough to live in; Altium DXP eats a gigabyte on its own, so starting and stopping the tool is painful as the page file starts kicking in.

Minor nit: the TrackPoint mouse will become uncalibrated if you lean on it lightly for too long, and the mouse will start gliding across the screen on its own. Fortunately, the driver is very intelligent and detects this condition so it’s short-lived.

This is a general fault of windows, but the 65k GDI resource limitation just sucks. Running out of GDI resources was the key source of reboots for me; Adobe tools and Altium DXP were notorious for leaking GDI objects, and unfortunately I use both heavily.

Bluescreens were rare, typically caused by either pulling out USB devices at inopportune times or problems occurring during standby, but one weirdness really got to me. Toward the end, if I walked around with the laptop, it would blue screen. This was because the hard drive shock detector would pause the drive, which would cause some application fault in windows that wasn’t expecting the pause. If I had to move around, I had to suspend the shock protection, which is sort of defeating the purpose of the protection. This wasn’t the case when I first started using my laptop. Why it developed this problem over time is a mystery to me.

Well, that’s my experience with the IBM T60p. I think my experience can be summarized by the fact that my new laptop is an IBM T61p — they won a repeat customer. The T61p is still settling in. I like the extra performance it brings, but it has some quirks of its own that I’m still learning to get used to. It’s funny, but every time I get a new laptop, I have to learn all of its little bugs and find ways to patch my behavior to avoid aggravating them. For example, I’ve trained myself to unplug my webcam before hitting the eject button on the minidock, because failing to do so causes a blue screen without fail. So far, none of the quirks have been so bad that I’ve given up on the machine. In addition to the webcam issue, I did have a spate of blue screens to deal with when I first got the laptop (mostly around minidock ejection and going into and out of standby), and I had to do some digging around in the Minidump logs to figure out exactly what behavior was triggering them. I wonder what other users do when this sort of thing happens to them?
