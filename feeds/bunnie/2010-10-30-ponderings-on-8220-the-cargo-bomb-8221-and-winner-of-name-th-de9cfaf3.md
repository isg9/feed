---
title: Ponderings on &#8220;The Cargo Bomb&#8221; (and Winner of Name that Ware October 2010)
url: https://www.bunniestudios.com/blog/2010/ponderings-on-the-cargo-bomb-and-winner-of-name-that-ware-october-2010/
published: "2010-10-30T23:44:54Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1359
---

# Ponderings on &#8220;The Cargo Bomb&#8221; (and Winner of Name that Ware October 2010)

The name that ware crowd does it again — guessed within the first few hours of being posted. Ryan Bavetta wins for being the first with the correct answer. email me to claim your prize!

Of course, I don’t have access to the ware itself so I must apply my judgment to the guesses, but I believe it’s fairly safe to say that it’s a Nokia 6120c or very closely related model (the entire 612x family has motherboards that are basically identical sans minor changes for specific regional or carrier variants; see the [wikipedia page for the Nokia 6120](http://en.wikipedia.org/wiki/Nokia_6120_classic) family).

I managed to dig up the original [service manual schematics](http://bunniestudios.com/blog/images/Schematics_6120c.pdf) for the Nokia 6120c. There are some very curious features about the preparation of the cargo bomb package. First of all, the phone motherboard only has two wires (plus perhaps a ground strap) attached to it. I’m presuming at least one of the wires is for a battery voltage, assuming the return current is going through the metal case via the middle screw.

If this were, for example, a trigger mechanism for something, then presumably the other wire is for the trigger signal.

What makes this a little bit odd, then, is the lack of an antenna. If you look at the schematics for the device, there is a set of four leaf connectors at the top of the motherboard, X7550, X7551, X7552, and X7555 (would be on the rear right side in the photo taken by the press), which need to come in touch with an antenna for any reception worth a damn. I don’t see evidence of an antenna attached to these from the press photo, and if there was it would be pretty close to the large ground plane presented by the metal case. The sensitivity of the radio would be fairly bad, making it unreliable at best as a remotely activated trigger.

One may presume that this is simply because the creator of this package was not skilled in electronics; if that’s the case then I feel a little bit safer since the “bad guys” don’t know how to build a reliable remote bomb trigger out of a cell phone.

However, another possibility is that the motherboard didn’t even have a SIM card in it, and as a result this is just a cheeseball version of the “alarm clock” that you would see in, for example, a “movie bomb”. If they simply attached a wire to the vibrator motor terminals or the ringer/speaker connector, and set a wake-up alarm a couple days later, this would function fairly decently as a time-delay device to activate some mechanism. It’s not hard to find a used mobile phone that doesn’t work as a phone, but still works well enough to set an alarm, although if I were looking for a simple mechanism to just act as a trigger I wouldn’t pick something that has an [IMEI](http://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity) (International Mobile Equipment Identity) or other serial numbers that can be traced through a supply chain. Then again, let’s hope that the “bad guys” aren’t smart enough to realize that mobile phones make poor event triggers if you were hoping for some kind of anonymity.

A little more browsing of the latest press releases note that there was a SIM card in the device, so presumably this was intended to receive a call to detonate the package. Glad to hear the sender of the package doesn’t know much about RF circuits and antennas. Granted, a phone can still receive a signal without an antenna, but the reliability would be poor; you’d need to be much closer to a base station so you have a high chance of failure in executing the plot. And SIM cards contain a wealth of traceable information. At the very least, someone has to call the phone to set off the trigger. If the phone is intercepted and the SIM card is put into a normal phone, the plotter would be unpleasantly surprised to find that it’s the FBI answering (and looking at your caller ID), instead of a bomb going off. Furthermore, scanning packages for suspicious devices becomes a lot easier, because you can just use a handheld RF scanner to look for radio waves in key frequency bands coming out of boxes that you would otherwise expect to be inert. In other words, a box with an active phone on the inside would advertise its presence in a detectable way to the outside world through its RF signature.

Of course, all wild speculation based on one low-res photo of a phone motherboard…
