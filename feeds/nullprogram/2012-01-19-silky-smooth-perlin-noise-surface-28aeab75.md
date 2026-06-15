---
title: Silky Smooth Perlin Noise Surface
url: https://nullprogram.com/blog/2012/01/19/
published: "2012-01-19T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/01/19/
---

# Silky Smooth Perlin Noise Surface

## [Silky Smooth Perlin Noise Surface](/blog/2012/01/19/)

January 19, 2012

nullprogram.com/blog/2012/01/19/

At work I’ve recently been generating [viewsheds](http://en.wikipedia.org/wiki/Viewshed) over [DTED](http://en.wikipedia.org/wiki/DTED) sets. Earlier this week I was asked to give an informal presentation on what I was doing. I wanted some terrain that demonstrated some key features, such as vision being occluded by hills of varying heights. Rather than search through the available DTED files for something good, I opted for generating my own terrain, using an old trick of mine: [my noise “cloud” generator](/blog/2007/11/20/). That’s a lesson in the usefulness of maintaining a blog. The useful things you learn and create are easy to revisit years later!

I generated some noise, looked at it with `surf()`, and repeated until I found something useful. ( *Update June 2012:* the function is called `perlin()` but it’s not actually Perlin noise.)

```
m = perlin(1024);
surf(m);

```

The generated terrain is really quite rough, so I decided to smooth it out by [convolving it with a 2-dimensional Gaussian kernel](/blog/2008/02/22/).

```
k = fspecial('gaussian', 9);
ms = conv2(m, k, 'same');

```

It still wasn’t smooth enough. So I repeated the process a bit,

```
for i = 1:10
    ms = conv2(ms, k, 'same');
end

```

Perfect! I used that for my presentation. However, I was having fun and decided to experiment more with this. I filtered it again another 1000 times and generated a `surf()` plot with a high-resolution colormap — the default colormap size caused banding.

```
colormap(copper(1024));
surf(ms, 'EdgeAlpha', 0);
axis('equal');

```

It produced this beautiful result!

[![](/img/noise/silk-perlin-surface-thumb.jpg)](/img/noise/silk-perlin-surface.jpg)

I think it looks like a photograph from a high-powered microscope, or maybe the turbulent surface of some kind of creamy beverage being stirred.

At work when I need something Matlab-ish, I use Octave about half the time and Matlab the other half. In this case, I was using Matlab. Octave doesn’t support the `EdgeAlpha` property, nor the `viewshed()` function that I needed for my work. Matlab currently makes much prettier plots than Octave.

- [octave](/tags/octave/)
- [math](/tags/math/)
- [media](/tags/media/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Silky%20Smooth%20Perlin%20Noise%20Surface) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Silky+Smooth+Perlin+Noise+Surface).

« [CSS Variables with Jekyll and Liquid](/blog/2011/12/16/)

» [Cartoon Liquid Simulation](/blog/2012/02/03/)
