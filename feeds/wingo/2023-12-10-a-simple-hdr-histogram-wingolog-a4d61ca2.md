---
title: a simple hdr histogram — wingolog
url: https://wingolog.org/archives/2023/12/10/a-simple-hdr-histogram
published: "2023-12-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/12/10/a-simple-hdr-histogram
---

# a simple hdr histogram — wingolog

## [a simple hdr histogram](/archives/2023/12/10/a-simple-hdr-histogram)

10 December 2023 9:27 PM

- [histogram](/tags/histogram)
- [telemetry](/tags/telemetry)
- [performance](/tags/performance)
- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [latency](/tags/latency)

Good evening! This evening, a note on high-dynamic-range (HDR) histograms.

### problem

How should one record garbage collector pause times?

A few options present themselves: you could just record the total pause time. Or, also record the total number of collections, which allows you to compute the average. Maximum is easy enough, too. But then you might also want the median or the p90 or the p99, and these percentile values are more gnarly: you either need to record all the pause times, which can itself become a memory leak, or you need to approximate via a histogram.

Let’s assume that you decide on the histogram approach. How should you compute the bins? It would be nice to have microsecond accuracy on the small end, but if you bin by microsecond you could end up having millions of bins, which is not what you want. On the other end you might have multi-second GC pauses, and you definitely want to be able to record those values.

Consider, though, that it’s not so important to have microsecond precision for a 2-second pause. This points in a direction of wanting bins that are *relatively* close to each other, but whose *absolute* separation can vary depending on whether we are measuring microseconds or milliseconds. You want approximately uniform precision over a high dynamic range.

### logarithmic binning

The basic observation is that you should be able to make a histogram that gives you, say, 3 significant figures on measured values. Such a histogram would count anything between 1230 and 1240 in the same bin, and similarly for 12300 and 12400. The gap between bins increases as the number of digits grows.

Of course computers prefer base-2 numbers over base-10, so let’s do that. Say we are measuring nanoseconds, and the maximum number of seconds we expect is 100 or so. There are about 230 nanoseconds in a second, and 100 is a little less than 27, so that gives us a range of 37 bits. Let’s say we want a precision of 4 significant base-2 digits, or 4 bits; then we will have one set of 24 bins for 10-bit values, another for 11-bit values, and so-on, for a total of 37 × 24 bins, or 592 bins. If we use a 32-bit integer count per bin, such a histogram would be 2.5kB or so, which I think is acceptable.

Say you go to compute the bin for a value. Firstly, note that there are some values that do not have 4 significant bits: if you record a measurement of 1 nanosecond, presumably that is just 1 significant figure. These are like the [denormals](https://en.wikipedia.org/wiki/Subnormal_number) in floating-point numbers. Let’s just say that recording a value *val* in \[0, 24-1\] goes to bin *val*.

If *val* is 24 or more, then we compute the *major* and *minor* components. The major component is the number of bits needed to represent *val*, minus the 4 precision bits. We can define it like this in C, assuming that *val* is a 64-bit value:

```
#define max_value_bits 37
#define precision 4
uint64_t major = 64ULL - __builtin_clzl(val) - precision;

```

The `64 - __builtin_clzl(val)` gives us the ceiling of the base-2 logarithm of the value. And actually, to take into account the denormal case, we do this instead:

```
uint64_t major = val < (1ULL << precision)
  ? 0ULL
  : 64ULL - __builtin_clzl(val) - precision;

```

Then to get the minor component, we right-shift *val* by *major* bits, unless it is a denormal, in which case the minor component is the value itself:

```
uint64_t minor = val < (1 << precision)
  ? val
  : (val >> (major - 1ULL)) & ((1ULL << precision) - 1ULL);

```

Then the histogram bucket for a given value can be computed directly:

```
uint64_t idx = (major << precision) | minor;

```

Of course, we would prefer to bound our storage, hence the consideration about 37 total bits in 100 seconds of nanoseconds. So let’s do that, and record any out-of-bounds value in the special last bucket, indicating that we need to expand our histogram next time:

```
if (idx >= (max_value_bits << precision))
  idx = max_value_bits << precision;

```

The histogram itself can then be defined simply as having enough buckets for all major and minor components in range, plus one for overflow:

```
struct histogram {
  uint32_t buckets[(max_value_bits << precision) + 1];
};

```

Getting back the lower bound for a bucket is similarly simple, again with a case for denormals:

```
uint64_t major = idx >> precision;
uint64_t minor = idx & ((1ULL << precision) - 1ULL);
uint64_t min_val = major
  ? ((1ULL << precision) | minor) << (major - 1ULL)
  : minor;

```

### y u no library

How many lines of code does something need to be before you will include it as a library instead of re-implementing? If I am honest with myself, there is no such limit, as far as code goes at least: only a limit of time. I am happy to re-implement anything; time is my only enemy. But strategically speaking, time too is the fulcrum: if I can save time by re-implementing over integrating a library, I will certainly hack.

The canonical library in this domain is [HdrHistogram](https://hdrhistogram.github.io/HdrHistogram/). But even the C port is thousands of lines of code! A histogram should not take that much code! Hence this blog post today. I think what we have above is sufficient. HdrHistogram’s documentation speaks in terms of base-10 digits of precision, but that is easily translated to base-2 digits: if you want 0.1% precision, then in base-2 you’ll need 10 bits; no problem.

I should of course mention that HdrHistogram includes an API that compensates for [coordinated
omission](https://groups.google.com/g/mechanical-sympathy/c/icNZJejUHfE/m/BfDekfBEs_sJ), but I think such an API is [straigtforward to build on
top of the basics](https://www.javadoc.io/static/org.hdrhistogram/HdrHistogram/2.1.12/org/HdrHistogram/AbstractHistogram.html#recordValueWithExpectedInterval-long-long-).

My code, for what it is worth, and which may indeed be buggy, is [over
here](https://github.com/wingo/whippet/blob/main/api/gc-histogram.h). But don’t re-use it: write your own. It could be much nicer in C++ or Rust, or any other language.

Finally, I would note that somehow this feels very basic; surely there is prior art? I feel like in 2003, Google would have had a better answer than today; alack. Pointers appreciated to other references, and if you find them, do tell me more about your search strategy, because mine is inadequate. Until then, gram you later!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)

### 2 responses

1. Hubert says:[11 December 2023 10:25 AM](#49569f98271e72f5eb44c0462e8385ef95add47c)

   Common C bug?

   If I remember well, equality comparisons in C require the use of == (rather than = for assignment). Among others:

   min *val = major ? (...) : minor;*

   *will need to become (I think; wouldn’t be an issue in Scheme):*

   *min* val == major ? (...) : minor;

   Yes?

2. Hubert says:[11 December 2023 10:30 AM](#7737ee1d0c3655b8b7b11a6519229bd14846d472)

   Wooooopsy!

   My bad! (not fully awake yet!) Sorry ...

Comments are closed.
