---
title: A Quick Look at the SoftIron OverDrive 1000
url: https://blog.regehr.org/archives/1465
published: "2017-01-31T14:26:17Z"
feed: regehr
guid: http://blog.regehr.org/?p=1465
---

# A Quick Look at the SoftIron OverDrive 1000

ARM processors are ubiquitous in mobile, but you won’t find a lot of developers using ARM-based boxes for their day-to-day work. Recently I needed some ARM machines for compiler work and got a few Raspberry Pi 3 boards, which seemed reasonable since these have four cores at 1.2 GHz. However, while these boards are far faster than the original Raspberry Pi, they’re still very painful for development work; the SD card interface seems to be a major bottleneck and also I would imagine that the 512 KB last-level-cache is not holding enough of the working set of a developer workload to be all that effective.

The obviously desirable ARM chip for a developer box is a [ThunderX2](http://www.cavium.com/ThunderX2_ARM_Processors.html), with up to 54 cores at 3 GHz, but these don’t seem to be available at all until later in 2017 and who knows when someone will get around to packaging these up in an affordable, developer-friendly box. The original ThunderX is available for servers but I didn’t find an inexpensive boxed-up version (there’s the [Gigabyte R270-T61](http://b2b.gigabyte.com/products/product-page.aspx?pid=5927#ov) that’s hard to even get a price on). Various fastish Qualcomm chips are also available on development boards but not as complete systems, as far as I could tell. I’ve done my time bringing up software environments on random development boards and am no longer interested in that.

Anyway, I settled on the [SoftIron OverDrive 1000](https://softiron.com/products/overdrive-1000/), which has pretty good specs: 8 GB of DDR4 RAM, 1 TB disk, and an [AMD A1120 processor](http://www.anandtech.com/show/9956/the-silver-lining-of-the-late-amd-opteron-a1100-arrival), an implementation of ARM’s [Cortex-A57](https://en.wikipedia.org/wiki/ARM_Cortex-A57) design. The cores run at 1.7 GHz and each pair of cores shares a 1 MB L2 cache, with 8 MB of L3 cache shared across all four cores. The box is headless but has a USB serial console, gigabit Ethernet, SATA, and USB interfaces available. At $600 the price is right. Here’s [the documentation](http://cdn.softiron.com/OD1000_QSGv1.pdf).

The OverDrive 1000 ships with 64-bit openSUSE installed. Though I hadn’t used its zypper package manager before, it is very easy and pleasant to interact with. My one gripe with the default configuration so far is that it ships with under a GB of swap space and when doing big compiles (and especially links, and even more especially links of debug builds) it’s easy to get zapped by the OOM killer. The disk is formatted with btrfs which does not support swapfiles. Their support was responsive but didn’t have a recipe for resizing the root partition without removing the hard drive and plugging it into a different machine, which I haven’t bothered to do yet.

Now let’s look at how fast this thing is. I’m not trying to be scientific here, just giving a general idea of what to expect from this box. Since I often sit around waiting for compilers, the benchmark is going to be LLVM 4.0 rc1 compiled using itself. The test input (program being compiled) is [Crypto++ 5.6.5](https://www.cryptopp.com/), which I chose since it compiles fairly quickly and doesn’t seem to have a lot of external dependencies. I compiled it with the `-DCRYPTOPP_DISABLE_ASM` flag to disable use of assembly language that might add compile-time differences across the platforms.

The processors I’m testing are just some that happened to be convenient. There’s a machine based on a Core i7-2600, a quad-core from 2011 running at 3.4 GHz, it has hyperthreading turned off. Another is based on an i7-6950X, a 10-core from 2016 running at 3.0 GHz, it has hyperthreading turned on. Finally there’s a Macbook Pro 2.2 GHz retina model from mid-2015, it has a Core i7-4770HQ, a quad-core, also with hyperthreading turned on.

The i7-2600 and the i7-6950X run hot: they are in the 100 W range. The i7-4770HQ is rated at 47 W but this includes the GPU. I’ll speculate that perhaps the power used by the CPU part is not that different from the 25 W used by the AMD A1120 (please leave a comment if you know more about this) (update: see [this comment](https://blog.regehr.org/archives/1465#comment-19070)).

First, build time using one core (i.e. make -j1):

OverDrive 1000390 sMacbook Pro177 si7-2600113 si7-6950X139 s

So the ARM chip is about 3.5x slower than the fastest of the Intel chips.

Second, compile times using 4 cores:

OverDrive 1000137 sMacbook Pro57 si7-260036 si7-6950X41 s

The OverDrive 1000 gets about a 2.8x speedup from four cores. Of course some of the non-linearity is due to sequential processing in the software build; when compiling other projects, I’ve seen more like a 3.5x speedup from using four cores.

Finally, compile times using all cores:

OverDrive 1000137 sMacbook Pro49 si7-260036 si7-6950X19 s

So here’s the worst case slowdown of the OverDrive 1000: it’s 7.2x slower than the big Intel chip, but it’s a pretty unfair comparison since that chip costs $1,600.

Overall, the OverDrive 1000 is an inexpensive and capable machine, but it isn’t going to compete performance-wise with Intel boxes at the same price point (for example, [here are the PCs you can buy at NewEgg for between $500 and $600](https://www.newegg.com/Product/ProductList.aspx?Submit=ENE&IsNodeId=1&N=100019096&LeftPriceRange=500%20600)). If you buy an OverDrive 1000, buy it because you want an ARMv8 machine that shows up ready to use and that isn’t an embedded systems toy like the Raspberry Pi family.
