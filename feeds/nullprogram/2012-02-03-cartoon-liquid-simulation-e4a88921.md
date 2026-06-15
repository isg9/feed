---
title: Cartoon Liquid Simulation
url: https://nullprogram.com/blog/2012/02/03/
published: "2012-02-03T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/02/03/
---

# Cartoon Liquid Simulation

## [Cartoon Liquid Simulation](/blog/2012/02/03/)

February 03, 2012

nullprogram.com/blog/2012/02/03/

**Update June 2013**: This program has been [ported to WebGL](/blog/2012/02/03/)!!!

The other day I came across this neat visual trick: [How to simulate liquid](http://www.patrickmatte.com/stuff/physicsLiquid/) (Flash). It’s a really simple way to simulate some natural-looking liquid.

- Perform a physics simulation of a number of circular particles.
- Render this simulation in high contrast.
- Gaussian blur the rendering.
- Threshold the blur.

![](/img/liquid/liquid-thumb.png)

I \[made my own version\]\[fun\] in Java, using [JBox2D](http://jbox2d.org/) for the physics simulation.

- [https://github.com/skeeto/fun-liquid](https://github.com/skeeto/fun-liquid)

For those of you who don’t want to run a Java applet, here’s a video demonstration. Gravity is reversed every few seconds, causing the liquid to slosh up and down over and over. The two triangles on the sides help mix things up a bit. The video flips through the different components of the animation.

It’s not a perfect liquid simulation. The surface never settles down, so the liquid is lumpy, like curdled milk. There’s also a lack of cohesion, since JBox2D doesn’t provide cohesion directly. However, I think I could implement cohesion on my own by writing a custom contact.

JBox2D is a really nice, easy-to-use 2D physics library. I only had to read the first two chapters of the [Box2D](http://box2d.org/) manual. Everything else can be figured out through the JBox2D Javadocs. It’s also available from the Maven repository, which is the reason I initially selected it. My only complaint so far is that the API doesn’t really follow best practice, but that’s probably because it follows the Box2D C++ API so closely.

I’m excited about JBox2D and I plan on using it again for some future project ideas. Maybe even a game.

The most computationally intensive part of the process *isn’t* the physics. That’s really quite cheap. It’s actually blurring, by far. Blurring involves [convolving a kernel](/blog/2008/02/22/) over the image — O(n^2) time. The graphics card would be ideal for that step, probably eliminating it as a bottleneck, but it’s unavailable to pure Java. I could have [pulled in lwjgl](/blog/2011/11/06/), but I wanted to keep it simple, so that it could be turned into a safe applet.

As a result, it may not run smoothly on computers that are more than a couple of years old. I’ve been trying to come up with a cheaper alternative, such as rendering a transparent halo around each ball, but haven’t found anything yet. Even with that fix, thresholding would probably be the next bottleneck — something else the graphics card would be really good at.

- [interactive](/tags/interactive/)
- [java](/tags/java/)
- [math](/tags/math/)
- [media](/tags/media/)
- [video](/tags/video/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Cartoon%20Liquid%20Simulation) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Cartoon+Liquid+Simulation).

« [Silky Smooth Perlin Noise Surface](/blog/2012/01/19/)

» [Lisp Let in GNU Octave](/blog/2012/02/08/)
