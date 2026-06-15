---
title: Winner, Name that Ware February 2024
url: https://www.bunniestudios.com/blog/2024/winner-name-that-ware-february-2024/
published: "2024-03-15T16:02:34Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=7011
---

# Winner, Name that Ware February 2024

[![](https://bunniefoo.com/ntw/ntw_feb_2024_a_sm.jpg)](https://bunniefoo.com/ntw/ntw_feb_2024_a.jpg)

The ware for February 2024 is the core of a [B&G 213 Masthead Wind Sensor](https://www.bandg.com/bg/type/instrument-sensors-and-transducers/wind-sensors/213-masthead-unit/), an instrument capable of reporting both wind speed and direction. Thanks again to FETguy and [Renew Computers](https://renewcomputers.com/) for the contribution! The coil on the left hand side is a brushless [resolver](https://en.wikipedia.org/wiki/Resolver_(electrical)), which determines the angle of the wind; the speed of the wind is detected by the pair of inductors on the right hand side.

One might assume the right hand coils are part of a switching power regulator (due to their shape and size), but, interestingly, they are connected with tiny traces, and there are no large capacitors nearby that could be used for filtering. Instead, it seems the coils are used to pick up the movements of magnets that would revolve around the assembly. Presumably these magnets would be attached to the shaft of the anemometer cup assembly, thus giving a read on wind speed.

Personally, I would have implemented something like this using a Melexis rotary position sensor chip, like the [MLX90324](https://www.melexis.com/en/product/MLX90324/Triaxis-mainstream-rotary-position-sensor-IC-Analog-PWM-SENT). These gems can determine the angle of a magnetically coupled axis to 10 bits precision over its entire rated temperature range. I’m guessing there must be something that prevents the use of hall-effect sensors in the application — not sure what, but it would be interesting to know why.

I was thinking I’d give the prize to anyone who pointed out the oddity of the inductors on the right hand side of the board, but nobody seemed to have noticed that. There was one poster, Anon, who did name the exact make and model, but the explanation didn’t do enough to convince me that it wasn’t imaged-searched first and then backfilled with some details. If I judged this incorrectly, I apologize. But, given a lack of satisfactory answers, I will say this month there is no winner.

Perhaps if I let the competition run till the end of the month I’d have more entries, but the competition post will also get progressively more buried under IRIS updates. Thanks for playing, and hopefully my Name that Ware subscribers aren’t too annoyed by the temporary shift in gears!
