---
title: 'An Intuitive Motor: IQ Control&#8217;s Serial-to-Position Module'
url: https://www.bunniestudios.com/blog/2018/an-intuitive-motor-iq-controls-serial-to-position-module/
published: "2018-03-19T12:49:38Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5215
---

# An Intuitive Motor: IQ Control&#8217;s Serial-to-Position Module

Back when I was a graduate student, my advisor [Tom Knight](https://en.wikipedia.org/wiki/Tom_Knight_(scientist)) bestowed upon me many excellent aphorisms. One of them was, “just wrap a computer around it!” – meaning, rather than expending effort to build more perfect systems, wrap imperfection-correcting computers around imperfect systems.

An everyday example of this is the noise-cancelling headphone. Headphones offer imperfect noise cancellation, but by “wrapping a computer around it” – adding one or more microphones and a computer in the from of a digital signal processor (DSP) – the headphones are able to measure the ambient noise and drive the headphones with the exact inverse of the noise, thus cancelling out the surrounding noise and creating a more perfect listening experience.

Although the principle has found its way rapidly into consumer goods, it’s been very slow to find its way onto the engineer’s workbench. It’s the case of the cobbler’s children having no shoes.

In particular, it’s long bothered me that motors are so dumb. Motors are typically large, heavy, costly, power-hungry, and riddled with small mechanical imperfections. In comparison, microcontrollers are tiny, cheap, power-efficient, and could run software that trims imperfections while improving efficiency to the point where the motor + microcontroller combo is a win over a dumb motor on almost every metric. So why aren’t we wrapping a computer around every motor and just calling it a day?

Then one day a startup called [IQ Motion Control](http://iq-control.com/) showed me a demo of their smart motor, the [IQ Position Module](https://www.crowdsupply.com/iq-motion-control/iq-motor-module), at [HAX in Shenzhen](https://hax.co/mentors/bunnie-huang/). My eyes instantly lit up – these guys have done it, and done it in a tasteful manner. This is the motor I’ve been waiting years for!

**Meet the IQ Position Module**

Simply put, the IQ Position Module is a brushless DC motor that talks serial and “thinks” at a higher level. I don’t have to design any complicated drive circuitry or buy a proprietary controller that talks some arcane or closed standard. I just plug an [FTDI cable](https://www.sparkfun.com/products/9717) into my laptop, hook up power, clone a [small git repo](https://github.com/bunnie/iqmotor-c) and I’m good to go.

Because of the microcontroller on the inside, the IQ Position Module can emulate a range of behaviors, from a simple stepper to a range of BLDC drive standards, but the real magic happens when you tell it where you want it to go and how fast, and it figures out the best way to get there.

“But wait”, you say, “my servos and brushed DC motors can do that just fine, I just control the pulse width!” This is true for crude and slow motion control applications, but if you really want to run at high speeds – like the ones achievable by a BLDC – you have to consider things like acceleration and deceleration profiles.

The video below shows what I mean. Here is a pre-production IQ Position Module that’s [being commanded](https://github.com/bunnie/iqmotor-c/blob/blog-post/simple_turn_demo.cpp#L39) to turn once in two seconds; then twice, three times, and finally ten times in two seconds. The motor can go even faster, but the figurine I attached on top isn’t balanced well enough to do that safely. Notice how the speed “ramps up” and back down again, so that the motor stops with the figurine in precisely the same position at the end of every cycle, regardless of how fast I commanded the motor to turn.

[https://bunniefoo.com/bunnie/iq\_turn\_demo.mp4](https://bunniefoo.com/bunnie/iq_turn_demo.mp4)

*That* is magic.

And here’s a snippet of the core [code used in the above demo](https://github.com/bunnie/iqmotor-c/blob/blog-post/simple_turn_demo.cpp), to give you an idea of how simple the API can be:

Just tell it where you want it, and by when — and the motor figures out an acceleration profile. Of course other parameters can be tweaked but the default behavior is reasonable enough!

**A Motor That’s Also an Input Device**

But wait! There’s more. Because this is a “direct drive” system, there’s no gears to shear. Anyone who has busted a geared servo motor by stalling or back-driving it knows what I mean. IQ Position Modules don’t have this problem. When you stop driving the IQ module – put it in a “coast” mode – it turns freely and without resistance.

This means the IQ motor doesn’t just “write” motion – it can “read” motion as well. Below is a video of a simple motion copy demo I cooked up in about an hour (including time spent refactoring the original API), where I implement [bidirectional read/write of motion](https://github.com/bunnie/iqmotor-c/blob/master/bidir_copy_demo.cpp) between two IQ Position Modules.

[https://bunniefoo.com/bunnie/iq\_motion\_copy\_demo\_lbr.mp4](https://bunniefoo.com/bunnie/iq_motion_copy_demo_lbr.mp4)

The ability to tolerate back-drive and also “go limp” is advantageous in robotics applications. Impact-oriented tasks — such as hammering a nail or kicking a ball — would rapidly degrade the teeth in a geared drive train. Furthermore, natural human motion incorporates the ability to go limp, such as the forward swing of a leg during walking. Finally, biological muscles are capable of applying a static force without changing position, such as when holding a cup on its sides without crushing it. Roboticists have developed a wide range of specialty actuators and techniques, including series elastic and variable stiffness actuators to address these scenarios. However, these mechanisms are often complicated and pricey.

The IQ Position Module’s lack of gearing means it’s back-drive tolerant, and it can apply an open-loop force without any risk of damage. This means you could, for example, use it to build a robot arm that can hammer a nail or pick up a cup. Robotic elements built using these would have far greater resilience to motion interference and impact forces than ones built using geared servos.

**Having Fun with the IQ Position Module**

While attending 34C3 back in December 2017, I managed to sit down for about an hour with my good friends [Prof. Nadya Peek](https://www.engr.washington.edu/facresearch/newfaculty/2017/NadyaPeek) and Ole Steinhauer, and we built a 2-axis robot arm that could do kinesthetic learning through keyframing, using nothing more than two IQ Position Modules, a Dunkin’ Donuts box, a bunch of schwag stickers we stole from the [FOSSASIA assembly](https://fossasia.org/), and the base plate of an old PS4 … because [fail0verflow](https://fail0verflow.com/blog/).

This was improvisational making at its best; we didn’t really plan the encounter so much as it emerged out of the chaos that is the Computer Congress. While Nadya was busy [cutting, folding, and binding the cardboard](http://cba.mit.edu/docs/papers/17.05.peek.pdf) into a 2-axis robot arm, Ole “joined” (#lötwat?!) together the power & serial connectors, and I furiously wrote the [code](https://github.com/bunnie/bldc-sandbox/blob/master/src/record_angle2.cpp) that would do the learning and playback — while also doing my best to polish off a couple beers. Nadya methodically built one motion axis first, and we tested it; satisfied with the result, she built and stacked a second axis on top. With just a bit of tweaking and prodding we managed to pull off the demo below:

[https://bunniefoo.com/bunnie/iq\_2axis\_robot.mp4](https://bunniefoo.com/bunnie/iq_2axis_robot.mp4)

It’s a little janky, but given the limited materials and time frame for execution, it hints at the incredible promise that IQ Position Modules hold.

So, if you’ve ever wanted to dabble in robotics or motion control, but have been daunted by control theory and arcane driver protocols (like I’ve been), check out the IQ Position Module. They are [crowdfunding now at CrowdSupply](https://www.crowdsupply.com/iq-motion-control/iq-motor-module). I backed their campaign to reserve a few more Position Modules for my lab – by wrapping a smart computer around a dumb motor, they’ve created a widget that lets me go from code to physical position and back with a minimal amount of wiring and an [accessible API](https://github.com/bunnie/iqmotor-c/blob/master/iqmotor.h).

Their current [funding campaign](https://www.crowdsupply.com/iq-motion-control/iq-motor-module) heavily emphasizes the capabilities of their motor as a “better BLDC” for the lucrative drone market, and I respect their wisdom in focusing their campaign message around a single, economically significant vertical. A cardinal sin of marketing revolutionary tech is to sell it as a [floor wax *and* a dessert topping](http://snltranscripts.jt.org/75/75ishimmer.phtml) — as painful as it may be, you have to pick just one message and push hard around it. However, I’m happy they are offering the IQ Position Module as [part of the campaign](https://www.crowdsupply.com/iq-motion-control/iq-motor-module), and enabling me to express my enthusiasm to the maker and robotics communities. I’ve waited too long to have a motor with this capability in my toolbox — finally, the cobbler’s children has shoes!
