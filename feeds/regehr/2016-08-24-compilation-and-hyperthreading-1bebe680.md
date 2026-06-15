---
title: Compilation and Hyperthreading
url: https://blog.regehr.org/archives/1416
published: "2016-08-24T17:01:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=1416
---

# Compilation and Hyperthreading

[Hyperthreading (HT)](http://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html) may or may not be a performance win, depending on the workload. I had poor luck with HT in the Pentium 4 era and ever since then have just disabled it in the BIOS on the idea that the kind of software that I typically wait around for—compilers and SMT solvers—is going to get hurt if its L1 and L2 cache resources are halved. This post contains some data about that. I’ll just start off by saying that for at least one combination of CPU and workload, I was wrong.

The benchmark is compilation of LLVM, Clang, and compiler-rt r279412 using Ninja on an [Intel i7-5820K](http://ark.intel.com/products/82932/Intel-Core-i7-5820K-Processor-15M-Cache-up-to-3_60-GHz), a reasonably modern but by no means new Haswell-E processor with six real cores. The compiler doing the compilation is a Clang 3.8.1 binary from the LLVM web site. The machine is running Ubuntu 14.04 in 64-bit mode.

[Full details about the machine are here](https://blog.regehr.org/archives/1225). As an inexpensive CPU workhorse I think it stands the test of time, though if you were building one today you would double (or more) the RAM and SSD sizes and of course choose newer versions of everything. I’m particularly proud of the crappy fanless video cards I found for these machines.

This is the build configuration command:

```
cmake -G Ninja -DLLVM_TARGETS_TO_BUILD=host -DLLVM_ENABLE_ASSERTIONS=1 -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Release ..

```

Then, on an otherwise idle machine, I built LLVM five times for each degree of parallelism up to 16, both with and without hyperthreading. Here are the results. Since the variation between runs was very low—a few seconds at worst—I’m not worrying about statistics.

![](https://blog.regehr.org/extra_files/hyperthreading/ht.png)

What can we take away from this graph? The main conclusion is that hyperthreading wins handily, reducing the best-case build time from 11.75 minutes to 10.04 minutes: an improvement of 1 minute and 42 seconds. Also, I had been worried that simply enabling HT would be detrimental since Linux would sometimes schedule two threads on the same real core when a different core was idle. The graph shows that either this happens only rarely or else it doesn’t hurt much when it happens. Overloading the system (forking more compilers than there are processors) hurts performance by just a very small amount. Of course, at some point the extra processes would use all RAM and performance would suffer significantly. Finally, the speedup is impressively close to linear until we start running more than one thread per core:

![](https://blog.regehr.org/extra_files/hyperthreading/ht-speedup.png)

I don’t know how much of the nonlinearity comes from resource contention and how much comes from lack of available parallelism.

Here are the [first](https://blog.regehr.org/extra_files/hyperthreading/ht.pdf) and [second](https://blog.regehr.org/extra_files/hyperthreading/ht-speedup.pdf) graphs as PDF.

Looking at the bigger picture, a huge amount of variation is possible in the compiler, the software being compiled, and the hardware platform. I’d be interested to hear about more data points if people have them.
