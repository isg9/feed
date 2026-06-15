---
title: A Seedable JavaScript PRNG
url: https://nullprogram.com/blog/2013/03/25/
published: "2013-03-25T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/03/25/
---

# A Seedable JavaScript PRNG

## [A Seedable JavaScript PRNG](/blog/2013/03/25/)

March 25, 2013

nullprogram.com/blog/2013/03/25/

In preparation for [my 7-day roguelike](/blog/2013/03/17/), I developed a [seedable pseudo-random number generator library](https://github.com/skeeto/rng-js). Half of it is basically a port of [BrianScheme’s PRNG](https://github.com/skeeto/brianscheme/blob/master/random.sch).

JavaScript comes with an automatically-seeded global uniform PRNG, Math.random(). This is suitable for most purposes, but if repeatability and isolation is desired — such as when generating a roguelike dungeon — it’s inadequate. I also wanted to be able to sample from different probability distributions, particularly the normal distribution and the exponential distribution.

The underlying number generator is the elegant RC4. Seeds are strings and anything else (except functions) is run through JSON.stringify(). Characters above code 255 are treated as two-byte values. If no seed is provided it will grab some of the available entropy for the job. To generate a uniform random double-precision value, 7 bytes (56 bits) are generated to account for the full 53-bit mantissa.

All other distributions sample from the uniform number generator, so bits are twiddled in only one place. Moreso, it means that Math.random() can be used as the core random number generator. My RC4 implementation is about 10x slower than V8’s Math.random(), so if all you care about is the probability distributions, not the seeding, then you could benefit from better performance. Just provide Math.random as the “seed”.

Here’s an example of it in action. I’m seeding it with an arbitrary object and generating six normally-distributed values. The output should be exactly the same no matter what JavaScript engine is used.

```
(function(array, n) {
    var rng = new RNG({foo: 'bar'});
    for (var i = 0; i < n; i++) {
        array.push(parseFloat(rng.normal().toFixed(4)));
    }
    return array;
}([], 6));
// => [0.807, -0.9347, -1.4543, -0.2737, 0.5064, -1.7342]

```

Provided probability distributions:

- Uniform
- [Normal](http://en.wikipedia.org/wiki/Normal_distribution)
- [Exponential](http://en.wikipedia.org/wiki/Exponential_distribution)
- [Poisson](http://en.wikipedia.org/wiki/Poisson_distribution)
- [Gamma](http://en.wikipedia.org/wiki/Gamma_distribution)

As far as the extras go, in my game I only ended up using the exponential distribution, for generating monster-spawning events. I intended to use the normal distribution for map generation, but, to save time, I used [rot.js](http://ondras.github.com/rot.js/hp/) for that purpose.

As far as testing goes, I basically just exported the output to GNU Octave so that I could eyeball the histogram and do some basic statistical checks. Everything *looks* reasonable, so I assume it’s implemented correctly. “That’s the problem with randomness. [You can never be sure](http://search.dilbert.com/comic/Random%20Number%20Generator).”

Using the same seed as above, here are some histograms of the first 10,000 samples for different probability distributions.

Uniform:

![](/img/plot/rngjs-uniform.png)

Normal:

![](/img/plot/rngjs-normal.png)

Exponential:

![](/img/plot/rngjs-exponential.png)

Gamma (mean = 4):

![](/img/plot/rngjs-gamma.png)

- [javascript](/tags/javascript/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20A%20Seedable%20JavaScript%20PRNG) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=A+Seedable+JavaScript+PRNG).

« [Applying Constructors in JavaScript](/blog/2013/03/24/)

» [JavaScript Fantasy Name Generator](/blog/2013/03/27/)
