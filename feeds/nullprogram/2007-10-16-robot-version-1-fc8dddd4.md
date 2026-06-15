---
title: Robot Version 1
url: https://nullprogram.com/blog/2007/10/16/
published: "2007-10-16T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2007/10/16/
---

# Robot Version 1

## [Robot Version 1](/blog/2007/10/16/)

October 16, 2007

nullprogram.com/blog/2007/10/16/

*Update: There is a [followup](/blog/2008/02/04/) post to this post.*

- [Full Album](https://picasaweb.google.com/106608599943434002866/RobotCompetition2007)

Here is what my team has been working on for the last couple weeks. The end goal for this robot is to escape a maze, collect blocks, and find a repository in which to drop those blocks. Someone suggested we call it Pac-man.

We added a third level to make more room for the batteries and extra sensors. The game board is 8x8 feet with a 4x8 foot maze.

Building a robot is an interesting experience, but a stressful one. Especially when you are doing it for a class. So many things could go wrong and you can spend hours tracking down a bad soldering job, which we once found inside an IR sensor. It was a poor manufacturing soldering job.

So, as of this writing, the robot uses 3 infrared (IR) sensors to look at walls and two wheel encoders for tracking the distance traveled by each wheel. You can see the disk encoder on the inside of the wheel in the second robot image. The robot uses 9 rechargeable nickel-metal hydride (NiMH) AA batteries: 5 for the Freescale 68HC12 micro-controller and sensors, and 4 for the continuous rotation servo motors. It is a competition, so I don’t want to give too many details at the moment in case another team is reading.

Right now it limps along in the maze and gets around for awhile before drifting into a wall. This will get fixed this weekend, as our grade depends on it. We just need to make better use of our sensors. I.e., it is a software issue now.

Eventually, I will put some code up here we used in the robot. It is all done in C, of course.

- [media](/tags/media/)
- [meatspace](/tags/meatspace/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Robot%20Version%201) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Robot+Version+1).

« [Java Animated Maze Generator and Solver](/blog/2007/10/08/)

» [Chess AI Idea](/blog/2007/10/24/)
