---
title: ARM Math Quirks on Raspberry Pi
url: https://blog.regehr.org/archives/793
published: "2012-09-12T17:31:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=793
---

# ARM Math Quirks on Raspberry Pi

Embedded processors can be relied upon to be a little quirky. Lately I’ve been playing around with the Raspberry Pi’s BCM2835 processor, which is based on the ARM1176JZF-S core. The “J” stands for Jazelle, a module that permits this processor to execute Java bytecodes directly. As far as I know there’s no open source support for using this, and my guess is that a JIT compiler will perform much better anyway, except perhaps in very RAM-limited systems. The “Z” indicates that this core supports TrustZone, which provides a secure mode switch so that secrets can be kept even from privileged operating system code. The “F” means the core has a floating point unit. Finally, “S” indicates a synthesizable core: one for which ARM licenses the Verilog or VHDL.

One thing I always like to know is how an embedded processor and its compiler cooperate to get things done; looking at the compiler’s output for some simple math functions is a good way to get started. I have a file of operations of the form:

```
long x00 (long x) { return x* 0; }
long x01 (long x) { return x* 1; }
long x02 (long x) { return x* 2; }
long x03 (long x) { return x* 3; }
...

```

Of course, multiplying by zero, one, and two are boring, but there’s a cute way to multiply by three:

> ```
> x03:
>     add r0, r0, r0, asl #1
>     bx lr
> ```

The computation here is r0 + (r0<<1), but instead of separate shift and add instructions, both operations can be done in a single instruction: ARM supports pre-shifting one operand “for free” as part of ALU operations. I had always considered this to be sort of a bizarre fallout from ARM’s decision to use fixed-size instruction words (gotta use all 32 bits somehow…) so it’s pretty cool to see the compiler putting the shifter to good use. Presumably GCC has a peephole pass that runs over the RTL looking for pairs of operations that can be coalesced.

Multiplying by four is just (r0<<2) and multiplying by five is r0 + (r0<<2). Multiply-by-six is the first operation that requires more than one ALU instruction and multiply-by-11 is the first one that requires an additional register. GCC (version 4.6.3 at -Ofast) doesn’t emit an actual multiply instruction until multiply-by-22, which is also the first multiplication that would require three ALU instructions. Perhaps unsurprisingly, people have put some thought into efficiently implementing multiply-by-constant using shifts and adds; [code at this page](http://spiral.ece.cmu.edu/mcm/gen.html) will solve instances of a pretty general version of this problem. Optimizing multiply seems to be one of those apparently simple problems that turns out to be deep when you try to do a really good job at it.

Turning multiply-by-constant into a sequence of simpler operations is satisfying, but is it productive? I did a bit of benchmarking and found that on the BCM2835, multiply-by-22 has almost exactly the same performance whether implemented using two instructions:

> ```
> x22:
>     mov r3, #22
>     mul r0, r3, r0
>     bx lr
> ```

or using three instructions:

> ```
> x22:
>     add r3, r0, r0, asl #2
>     add r0, r0, r3, asl #1
>     mov r0, r0, asl #1
>     bx lr
> ```

When a `mul` instruction can be replaced by fewer than three add/subtract/shift instructions, the substitution is profitable. In other words, GCC is emitting exactly the right code for this platform. This is perhaps slightly surprising since benchmarking on any specific chip often reveals that the compiler is somewhat suboptimal since it was tuned for some older chip or else because it makes compromises due to targeting some wide variety of chips.

Next I looked at integer division. Implementing divide-by-constant using multiplication is a pretty well-known trick—see Chapter 10 of [Hacker’s Delight](http://www.hackersdelight.org/)—so I won’t go into it here. On the other hand, division by non-constant held a bit of a surprise for me: it turns out the ARM1176JZF-S has no integer divide instruction. I asked a friend who works at ARM about this and his guess (off-the-cuff and not speaking for the company, obviously) is that when the ARM11 core was finalized more than 10 years ago, chip real estate was significantly more expensive than it is now, making a divider prohibitive. Also, division probably isn’t a big bottleneck for most codes.

If we take source code like this, where `ccnt_read()` is from my [previous post](https://blog.regehr.org/archives/794):

```
volatile int x, y, z;

unsigned time_div (void)
{
  unsigned start = ccnt_read ();
  z = x / y;
  unsigned stop = ccnt_read ();
  return stop - start;
}

```

The compiler produces:

> ```
> time_div:
>     stmfd sp!, {r4, lr}
>     mrc p15, 0, r4, c15, c12, 1
>     ldr r3, .L2
>     ldr r0, [r3, #0]
>     ldr r3, .L2+4
>     ldr r1, [r3, #0]
>     bl __aeabi_idiv
>     ldr r3, .L2+8
>     str r0, [r3, #0]
>     mrc p15, 0, r0, c15, c12, 1
>     rsb r0, r4, r0
>     ldmfd sp!, {r4, pc}
> ```

On average, for random arguments, the cost of the `__aeabi_idiv` call is 26 cycles, but it can be as few as 15 and as many as 109 (these results aren’t the raw timer differences; they have been adjusted to account for overhead). An example of arguments that make it execute for 109 cycles is -190405196 / -7. The C library code for integer divide is similar to the [code described here](http://me.henri.net/fp-div.html). Its slow path requires three cycles per bit for shift-and-subtract, and this is only possible due to ARM’s integrated shifter. Additionally, there’s some overhead for checking special cases. In contrast, a hardware divide unit should be able to process one or two bits per cycle.
