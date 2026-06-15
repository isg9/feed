---
title: Battery Packs and Defects
url: https://www.bunniestudios.com/blog/2008/battery-packs-and-defects/
published: "2008-01-07T09:21:11Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=220
---

# Battery Packs and Defects

Recently, someone on the chumby forum noted that the [Energizer ER-PHOTO battery pack](http://www.digiframes.com/related/0-4522/ER-PHOTO.html) works with the chumby. The ER-PHOTO is a handy little device that essentially emulates a 12V DC wallwart with a pass-through mode, so you can continue to use whatever is plugged into the battery pack while it charges. There’s a lot of devices out there that run off of 12V and use that classic 5.5mm DC barrel jack, so they are certainly handy to have around. As you can see in the photo below, the pack consists of 3x 1800mAh Li-Ion cells, which gives a nominal capacity of 20 Wh; of course, the step-up regulator is probably only 90% efficient or so, and the circuitry in the pack is fairly simple, so maybe it lacks the electronics to safely milk the last drop out of the batteries, resulting in a reduced delivered capacity.

[Scott Janousek](http://www.scottjanousek.com/blog/2008/01/04/chumby-runs-as-a-portable-device-with-a-simple-battery-mod/) has a nice little writeup about how you can install the battery pack inside a chumby, although I’d be more than a little bit wary about doing what he’s written up–he’s taken the raw Lithium Ion cells out of their protective plastic housing. The electrical tape wrapping won’t provide adequate protection against puncture or impact (which is possible if you’re carrying around a chumby and you drop it). This can lead to [cells catching fire](http://computer.howstuffworks.com/dell-battery-fire.htm) in a way that can’t be put out easily (only class D fire extinguishers work on these fires, and many homes and small offices have only type ABC extinguishers).

I had a “near fire” experience once where my electric Braun shaver developed a loose connection to the rechargeable cell (it’s cold-welded on, and the continuous vibration of my luggage being dragged over cobblestones in Italy eventually wore the metal tab down) so that the charging circuitry saw a substantial series resistance in its path–my guess is it was trying to do current mode charging without paying attention to the voltage–and the shaver got extremely hot and started to carbonize the potting glue used around the components. I ripped the shaver apart and sprayed the battery down with freeze-spray to keep it from going into thermal runaway. I’ve also had a few Dell Latitude battery packs get scary-hot during charging–hot potato hot, where I had to juggle it from hand to hand while I ran it to the freezer to cool it down.

At any rate, I encountered a surprising defect in an Energizer battery pack the other day. One of the folks at the chumby office got one to evaluate, charged it, but it didn’t seem to work. So I took it apart, and lo and behold, the battery terminals weren’t soldered into the motherboard. Click on the photo for a larger version of the picture, to see the defect clearly.

[![](http://bunniestudios.com/blog/images/energizer_drytabs_sm.jpg)](http://bunniestudios.com/blog/images/energizer_drytabs.jpg)

I’m not quite sure how this got through QA–almost certainly there is a 100% unit test at the end of the line. Maybe the tabs made just enough contact for it to pass final test, and then vibration during shipping displaced them. It’s a very scary defect, however, as loose wires in battery packs can lead to hazardous conditions.

I can definitely see how a bug like this can happen in the process; I can just imagine two operators in China sitting next to each other, one of them stuffing the terminals and the other soldering them in, and then during shift change one of the stuffed but unsoldered devices gets mixed in with the soldered batch. Most factories put controls in place to try to avoid these kinds of process omissions–but mistakes do happen. To be clear, we all live in glass houses, as mistakes sometimes happen when building chumbys as well. Whenever I see a teardown of a chumby posted on the net, I take a keen look at the photos to see if all of the procedures were followed during the construction of the chumby device.
