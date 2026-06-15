---
title: Profiling — wingolog
url: https://wingolog.org/archives/2005/10/28/profiling
published: "2005-10-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/10/28/profiling
---

# Profiling — wingolog

## [Profiling](/archives/2005/10/28/profiling)

28 October 2005 3:53 PM

- [python](/tags/python)

**Python**

For some reason Flumotion is running at 100% cpu when starting components. It didn't use to do this, and it doesn't really make much sense -- the rate of data input is limited by the capture devices, and a certain time's worth of data has to accumulate to fill all of the relevant buffers in the encoders, muxers, and streamers.

But I have no clue where this CPU usage is. I tried pressing control-C occasionally and getting backtraces in GDB, but the normal methods aren't working.

In the end this was all due to not having an idea of where Flumotion spends its time. I wanted to profile, but didn't want to use an instrumenting profiler, and there didn't exist a sampling profiler for Python. So I ported the [one I hacked on for Guile](//wingolog.org/archives/2004/06/18/post-advogato) to python; it's available [here](//wingolog.org/pub/statprof.py).

Usage goes like this:

```
>>> import statprof
>>> statprof.start()
>>> import test.pystone; test.pystone.pystones()
(1.3200000000000001, 37878.78787878788)
>>> statprof.stop()
>>> statprof.display()
  %   cumulative      self
 time    seconds   seconds  name
 23.01      1.36      0.31  pystone.py:79:Proc0
 15.04      0.60      0.20  pystone.py:133:Proc1
 11.50      0.16      0.16  pystone.py:45:__init__
 10.62      0.14      0.14  pystone.py:208:Proc8
  7.96      0.16      0.11  pystone.py:229:Func2
  7.96      0.11      0.11  pystone.py:221:Func1
  6.19      0.12      0.08  pystone.py:160:Proc3
  5.31      0.07      0.07  pystone.py:203:Proc7
  3.54      0.20      0.05  pystone.py:53:copy
  2.65      0.05      0.04  pystone.py:184:Proc6
  2.65      0.04      0.04  pystone.py:149:Proc2
  1.77      0.02      0.02  pystone.py:170:Proc4
  0.88      0.01      0.01  pystone.py:177:Proc5
  0.88      0.01      0.01  pystone.py:246:Func3
  0.00      1.36      0.00  pystone.py:67:pystones
  0.00      1.36      0.00  :1:?
---
Sample count: 113
Total time: 1.360000 seconds

```

There's lots of info in `help(statprof)`. Also it requires the itimer extension from [http://www.cute.fi/~torppa/py-itimer/](http://www.cute.fi/~torppa/py-itimer/). Just unpack the tarball and run `sudo python setup.py install`.

It's not very well tested at this point, but it gives similar results as the stock profiler (10-20 times faster though). It's of course not the same, because an instrumenting profiler unfairly penalizes procedure calls; while they do have a cost it is much less than their cost as measured by the stock profiler or hotshot.

I'd be interested in hearing about bugs in it for the next few weeks; after that it will probably slip from my mind though. (This is of course the natural fate of profilers, to bitrot. I have a larger rant about this for later.)

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)
- [object closure and the negative specification](/archives/2008/04/22/object-closure-and-the-negative-specification)
- [eeeevil](/archives/2007/11/28/eeeevil)

Comments are closed.
