---
title: High-Resolution Timing on the Raspberry Pi
url: https://blog.regehr.org/archives/794
published: "2012-09-10T03:58:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=794
---

# High-Resolution Timing on the Raspberry Pi

Just to be clear, this post is about measuring the times at which events happen. Making things happen at specific times is a completely separate (and much harder) problem.

The `clock_gettime()` function (under Raspbian) gives results with only microsecond resolution and also requires almost a microsecond to execute. This isn’t very helpful when trying to measure short-duration events. What we want is access to the CPU’s cycle counter, which gives closer to nanosecond resolution and has low overhead. This is easy to accomplish:

```
static inline unsigned ccnt_read (void)
{
  unsigned cc;
  asm volatile ("mrc p15, 0, %0, c15, c12, 1" : "=r" (cc));
  return cc;
}

```

The problem is that if you call this code from user mode, your process will die due to executing an illegal instruction. By default, user mode does not get to read the cycle count register. To change this, we need a silly little LKM:

```
#include <linux/module.h>
#include <linux/kernel.h>

/*
 * works for ARM1176JZ-F
 */

int init_module(void)
{
  asm volatile ("mcr p15,  0, %0, c15,  c9, 0\n" : : "r" (1));
  printk (KERN_INFO "User-level access to CCR has been turned on.\n");
  return 0;
}

void cleanup_module(void)
{
}

```

After the `insmod` call, the cycle count register will be accessible. For all I know there’s a good reason why this access is disabled by default, so please think twice before using this LKM on your security-critical Raspberry Pi. ( **UPDATE:** See pm215’s comment below, but keep in mind that if a local user wants to DOS your RPi board, there are many other ways to accomplish this.)

Annoyingly, the Raspbian folks have not yet released a kernel headers package for the current kernel (3.2.27+). Also, an LKM compiled against older headers will fail to load. However, [this thread](http://www.raspberrypi.org/phpBB3/viewtopic.php?t=15787&p=165638) contains a link to some updated headers.

[Here’s a tarball](https://blog.regehr.org/extra_files/raspbian-ccr.tar.gz) containing the code from this post and also a compiled LKM for the 3.2.27+ Raspbian kernel.

I’m writing this up since cycle counters are awesome and also stupid problems like the missing kernel headers made it take an embarrassing amount of time for me to get this going. There’s a lot of ARM timer code out there but all of it that I found is either intended for kernel mode or else fails to work on an ARM11 chip. I actually had to read [the manual](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0301h/DDI0301H_arm1176jzfs_r0p7_trm.pdf) to make this work.

The solution here is the best one that doesn’t involve modifying the kernel. A better approach would be to expose this functionality through /proc.
