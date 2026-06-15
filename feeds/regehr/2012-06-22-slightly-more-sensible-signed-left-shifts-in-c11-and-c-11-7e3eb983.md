---
title: Slightly More Sensible Signed Left-Shifts in C11 and C++11
url: https://blog.regehr.org/archives/738
published: "2012-06-22T02:42:03Z"
feed: regehr
guid: http://blog.regehr.org/?p=738
---

# Slightly More Sensible Signed Left-Shifts in C11 and C++11

Left-shift of signed integers in C99, C11, and C++11 is difficult to use because shifting a 1 bit into or past the sign bit (assuming two’s complement, of course) is an undefined behavior. Many medium and large C and C++ programs do this. For example, many codes use `1<<31` for `INT_MIN`. [IOC](http://embed.cs.utah.edu/ioc/) can detect this problem, but it happens so commonly, and developers are so uninterested in hearing about it, that we added command line flags that tell IOC to not check for it, in order to avoid annoying people.

Today a reader pointed me to [this active issue](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3367.html#1457) that the C++ standards committee is evaluating. Basically the proposal is to make it permissible to shift a 1 into the sign bit, but not past it. The effect will be to un-break a significant number of C/C++ programs that currently execute undefined behavior when compiled as C99, C11, or C++11. Issue 1457 is marked as “ready,” meaning (as far as I can tell) that it stands a good chance of being ratified and incorporated into future versions of the C11 and C++11 standards. This seems like a step in the right direction.
