---
title: Verifying a Driver
url: https://blog.regehr.org/archives/280
published: "2010-10-14T03:43:33Z"
feed: regehr
guid: http://blog.regehr.org/?p=280
---

# Verifying a Driver

Last week my student Jianjun Duan gave a talk about his PhD work at the [Workshop for Systems Software Verification](http://www.usenix.org/events/ssv10/) in Vancouver. Although preliminary — he verified only three small functions — this work is pretty cool. Jianjun started with [a model for the ARM instruction set](http://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-545.pdf) in [HOL4](http://hol.sourceforge.net/) and augmented it so that memory-mapped hardware peripherals could be plugged into it. The basic idea is that real systems are not just CPU+memory. This is obvious, but almost all existing results on formal verification of software ignore devices. This doesn’t work for embedded software that is closely tied to dozens of hardware devices integrated into a system on chip. Jianjun proved some basic sanity results like “if you have a system that works, it still works after plugging in a new hardware device.” Once you are dealing with a theorem prover, even simple results like this become nontrivial due to the extraordinary level of detail that is required.

The case study in this paper was to:

1. Formalize one of the UART (serial port) devices found on the [NXP LPC2129](http://ics.nxp.com/support/documents/microcontrollers/pdf/user.manual.lpc2109.lpc2114.lpc2119.lpc2124.lpc2129.lpc2194.lpc2210.lpc2212.lpc2214.lpc2220.lpc2290.lpc2292.lpc2294.pdf), a microcontroller built around the ARM7TDMI CPU
2. After plugging the UART model into the ARM model, prove full correctness for an open-source device driver that was compiled to ARM assembly from C

“Full correctness” means something like:

> The driver terminates, correctly transmits the characters it is asked to transmit, correctly receives characters that arrive over the wire, respects memory safety, and never puts the UART into a bad state.

Perhaps the coolest thing about Jianjun’s proofs are that they include timing properties. For example, the proof does not work if the UART’s clock divider is not set high enough. Reasoning about timing made the proofs a lot harder, even for a very simple timing model (which is reasonable for this class of CPU). The [slides from the talk](http://www.cs.utah.edu/~regehr/papers/ssv10-slides.pdf) give a pretty good idea of what is going on, and also the [paper is here](http://www.cs.utah.edu/~regehr/papers/ssv10.pdf).
