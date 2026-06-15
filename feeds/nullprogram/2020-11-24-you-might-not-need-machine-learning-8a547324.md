---
title: You might not need machine learning
url: https://nullprogram.com/blog/2020/11/24/
published: "2020-11-24T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/11/24/
---

# You might not need machine learning

## [You might not need machine learning](/blog/2020/11/24/)

November 24, 2020

nullprogram.com/blog/2020/11/24/

*This article was discussed [on Hacker News](https://news.ycombinator.com/item?id=25196574).*

Machine learning is a trendy topic, so naturally it’s often used for inappropriate purposes where a simpler, more efficient, and more reliable solution suffices. The other day I saw an illustrative and fun example of this: [Neural Network Cars and Genetic Algorithms](https://www.youtube.com/watch?v=-sg-GgoFCP0). The video demonstrates 2D cars driven by a neural network with weights determined by a generic algorithm. However, the entire scheme can be replaced by a first-degree polynomial without any loss in capability. The machine learning part is overkill.

[![](/img/screenshot/racetrack.jpg)](https://nullprogram.com/video/?v=racetrack)

Above demonstrates my implementation using a polynomial to drive the cars. My wife drew the background. There’s no path-finding; these cars are just feeling their way along the track, “following the rails” so to speak.

My intention is not to pick on this project in particular. The likely motivation in the first place was a desire to apply a neural network to *something*. Many of my own projects are little more than a vehicle to try something new, so I can sympathize. Though a professional setting is different, where machine learning should be viewed with a more skeptical eye than it’s usually given. For instance, don’t use active learning to select sample distribution when a [quasirandom sequence](http://extremelearning.com.au/unreasonable-effectiveness-of-quasirandom-sequences/) will do.

In the video, the car has a limited turn radius, and minimum and maximum speeds. (I’ve retained these contraints in my own simulation.) There are five sensors — forward, forward-diagonals, and sides — each sensing the distance to the nearest wall. These are fed into a 3-layer neural network, and the outputs determine throttle and steering. Sounds pretty cool!

![](/img/diagram/racecar.svg)

A key feature of neural networks is that the outputs are a nonlinear function of the inputs. However, steering a 2D car is simple enough that **a linear function is more than sufficient**, and neural networks are unnecessary. Here are my equations:

```
steering = C0*input1 - C0*input3
throttle = C1*input2

```

I only need three of the original inputs — forward for throttle, and diagonals for steering — and the driver has just two parameters, `C0` and `C1`, the polynomial coefficients. Optimal values depend on the track layout and car configuration, but for my simulation, most values above 0 and below 1 are good enough in most cases. It’s less a matter of crashing and more about navigating the course quickly.

The lengths of the red lines below are the driver’s three inputs:

These polynomials are obviously much faster than a neural network, but they’re also easy to understand and debug. I can confidently reason about the entire range of possible inputs rather than worry about a trained neural network [responding strangely](https://arxiv.org/abs/1903.06638) to untested inputs.

Instead of doing anything fancy, my program generates the coefficients at random to explore the space. If I wanted to generate a good driver for a course, I’d run a few thousand of these and pick the coefficients that complete the course in the shortest time. For instance, these coefficients make for a fast, capable driver for the course featured at the top of the article:

```
C0 = 0.896336973, C1 = 0.0354805067

```

Many constants can complete the track, but some will be faster than others. If I was developing a racing game using this as the AI, I’d not just pick constants that successfully complete the track, but the ones that do it quickly. Here’s what the spread can look like:

If you want to play around with this yourself, here’s my C source code that implements this driving AI and [generates the videos and images
above](/blog/2017/11/03/):

**[aidrivers.c](https://github.com/skeeto/scratch/blob/master/aidrivers/aidrivers.c)**

Racetracks are just images drawn in your favorite image editing program using the colors documented in the source header.

- [ai](/tags/ai/)
- [c](/tags/c/)
- [media](/tags/media/)
- [compsci](/tags/compsci/)
- [video](/tags/video/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20You%20might%20not%20need%20machine%20learning) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=You+might+not+need+machine+learning).

« [Improving on QBasic's Random Number Generator](/blog/2020/11/17/)

» [State machines are wonderful tools](/blog/2020/12/31/)
