---
title: Why Take an Embedded Systems Course?
url: https://blog.regehr.org/archives/195
published: "2010-06-29T00:06:11Z"
feed: regehr
guid: http://blog.regehr.org/?p=195
---

# Why Take an Embedded Systems Course?

Embedded systems are special-purpose computers that users don’t think of as computers. Examples include cell phones, traffic light controllers, and programmable thermostats. In earlier posts I argued why any computer scientist should take [a compilers course](https://blog.regehr.org/archives/169) and [an operating systems course](https://blog.regehr.org/archives/164). These were easy arguments to make since these areas are core CS: all graduates are expected to understand them. Embedded systems, on the other hand, often get little respect from mainstream computer scientists. So why should you take a course on them?

# Most Computers are Embedded

Around 99% of all processors manufactured go into embedded systems.  In 2007 alone, 2.9 billion chips based on the ARM processor architecture were manufactured; essentially all of these were used in embedded applications. These processors live in your car, appliances, and toys; they are scattered throughout our buildings; they are critical to efficient operation of the infrastructure providing transportation, water, and power. More and more the world depends on embedded systems; as a technical expert, it’s useful to understand how they work. The market for desktop computers is pretty much saturated; the embedded world is growing and looks to continue to do so as long as people continue to find it valuable to place computation close to the world.

# Embedded Programming Can Be Fun and Accessible

[Make Magazine](http://makezine.com/) has done a fantastic job popularizing embedded systems projects. Lego Mindstorms, Arduinos, and the like are not that expensive and can be used to get a feel for embedded programming.  Controlling the physical world is addictive and fun; head over to [Hack A Day](http://hackaday.com/) and search for “paintball sentry,” I defy any nerd to tell me with a straight face that this stuff is not totally cool. This spring I heard a fantastic talk by Sebastian Thrun about his team’s winning effort in the DARPA Grand Challenge ( [some of his presentation materials are online](http://robots.stanford.edu/talks/stanley/)).

# Embedded is Different

Monitoring and controlling the physical world is very different from other kinds of programming. For example, instead of nice clean discrete inputs, you may find yourself dealing with a stream of noisy accelerometer data. When controlling a motor, your code suddenly has to take the momentum of an actual piece of metal into account; if you’re not careful, you might break the hardware or burn out the driver chip. Similarly, robots live in a bewildering world of noisy, conflicting sensor inputs; they never go quite in a straight line or return to the angle they started at. Solving all of these problems requires robust methods that are very different from the algorithmically-flavored problems we often solve in CS.

# Embedded Makes You Solve Hard Problems

Difficult embedded programming problems force you to create highly reliable systems on constrained platforms. The software is concurrent (often using both interrupts and threads), must respect timing constraints from hardware devices and from the outside world, and must gracefully deal with a variety of error conditions. In summary, many of the hardest programming problems are encountered all at once.

Debugging facilities may be lacking. In the worst case, even the humble printf() is not available and you’ll be debugging using a logic analyzer and perhaps some LEDs. It brings me joy every year to sit a collection of CS students down in a lab full of logic analyzers; at first most of them are greatly confused, but by the end of the semester people are addicted to (or at least accustomed to) the ability to look at a real waveform or measure a pulse that lasts well under a microsecond.

Modern high-level programming environments are great, but they provide an awful lot of insulation from the kinds of bare-metal details that embedded programmers deal with all the time. People like me who learned to program during the 1980s have a reasonable intuition for what you can accomplish in 1 KB of RAM. On the other hand, programmers raised on Java do not. I’ve helped students working on a research project in sensor networks where their program didn’t work because it allocated more than a thousand times more RAM than was available on the platform they were trying to run on. Many years ago I spent far too long debugging problems that came from calling a printf() variant that allocated about 8 KB of stack memory from a thread that had 4 KB of stack.

All of these difficulties can be surmounted through careful design, careful implementation, and other techniques. By learning these things, students acquire valuable skills and thought processes that can also be applied to everyday programming.

# Embedded is in Demand

I don’t have good data supporting this, but anecdotally the kind of student who does well in an embedded systems course is in high demand. A lot of CS graduates understand only software; a lot of EE and ME graduates can’t program their way out of a paper bag. Students who combine these skill sets — regardless of what department gives them a degree — are really valuable.
