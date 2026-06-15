---
title: Isolation with a Very Small TCB
url: https://blog.regehr.org/archives/565
published: "2011-08-04T19:21:10Z"
feed: regehr
guid: http://blog.regehr.org/?p=565
---

# Isolation with a Very Small TCB

A basic service provided by most computer systems is isolation between different computations. Given reliable isolation we can safely run programs downloaded from the web, host competing companies’ sites on the same physical server, and perhaps even run your car’s navigation or entertainment software on the same processor that is used to control important system functions.

Other factors being equal, we’d like an isolation implementation to have a trusted computing base (TCB) that is as small as possible. For example, let’s look at the TCB for isolating Javascript programs from different sites when we browse the web using Firefox on a Linux box. It includes at least:

- pretty much the entire Firefox executable including all of the libraries it statically or dynamically links against (even unrelated code is in the TCB due to the lack of internal isolation in a C/C++ program)
- the Linux kernel including all kernel modules that are loaded
- GCC and G++
- some elements of the hardware platform (at least the processor, memory controller, memory modules, and any peripherals that use DMA)

A deliberate or accidental malfunction of any of these elements may subvert the isolation guarantee that Firefox would like to provide. This is a large TCB, though probably somewhat smaller than the corresponding TCB for a Windows machine. Microkernel-based systems potentially have a much smaller TCB, and a machine-verified microkernel like [seL4](http://ertos.nicta.com.au/research/sel4/) removes even the microkernel from the TCB. Even so, the seL4 TCB is still nontrivial, containing for example an optimizing compiler, the substantially complicated specification of the microkernel’s functionality, a model of the hardware including its MMU, and the hardware itself.

My student Lu Zhao’s PhD work is trying to answer the question: How small can we make the trusted ![](https://blog.regehr.org/wp-content/uploads/2011/08/model.png)

computing base for an isolation implementation? The target for this work is ARM-based microcontrollers that are large enough to perform multiple tasks, but way too small to run Linux (or any other OS that uses memory protection). Lu’s basic strategy is to implement software fault isolation (SFI). His tool rewrites an ARM executable in order to enforce two properties: memory safety and control flow integrity. We’re using a weak form of memory safety that forces all stores to go into a memory region dedicated to that task. Control flow integrity means that control never escapes from a pre-computed control flow graph. This boils down to rewriting the binary to put a dynamic check in front of every potentially dangerous operation. Using the excellent [Diablo](http://diablo.elis.ugent.be/) infrastructure, this was pretty easy to implement. I won’t go into the details but [they can be found in a paper we just finished](http://www.cs.utah.edu/~regehr/papers/emsoft11.pdf).

The trick to making Lu’s work have a small TCB was to remove the binary rewriter from the TCB by proving that its output has the desired properties. To do this, he needed to create mathematical descriptions of memory safety and control flow integrity, and also to borrow an [existing mathematical description of the ARM ISA](http://www.cl.cam.ac.uk/~acjf3/arm/). Reusing the ARM model was ideal because it has been used by several previous projects and it is believed to be of very high quality. Automating the construction of a non-trivial, machine-checked proof is not so easy and Lu spent a lot of time engineering proof scripts. The saving grace here is that since we control where to add safety checks, the proof is in some sense an obvious one — the postcondition of the check provides exactly the property needed to establish the safety of the potentially-unsafe operation the comes after the check.

The TCB for Lu’s isolation solution contains the [HOL4 theorem prover](http://hol.sourceforge.net/), his definitions of memory safety and control flow integrity (less than 60 lines of HOL), the model of the ARM ISA, and of course the ARM processor itself. All of these elements besides the processor are of quite manageable size. This work currently has two major drawbacks. First, the proofs are slow to construct and the automated proving part only succeeds for small executables. Second, overhead is far higher than it is for a state of the art sandboxing system such as [Native Client](http://code.google.com/p/nativeclient/). In principle the theorem proving infrastructure will be a powerful tool for optimizing sandboxed executables — but we haven’t gotten to that part of the work yet.

As far as we can tell, there’s not much more that can be removed from the TCB. Lu’s SFI implementation is the first one to do full formal verification of its guarantees for real executables. An important thing I learned from working with Lu on this project is that dealing with a mechanical theorem prover is a bit like having kids in the sense that you can’t approach it casually: you either go whole-hog or avoid it entirely. Perhaps in the future the theorem proving people will figure out how to make their tools accessible to dabblers like me.
