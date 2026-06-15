---
title: Testing with Small Capacities
url: https://blog.regehr.org/archives/1138
published: "2014-04-27T14:12:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1138
---

# Testing with Small Capacities

The goal of testing is to expose bugs. To find a bug we must drive the software under test into a part of its state space where the bug becomes evident, for example by causing a crash. Many bugs lurk in dark corners of the state space that are ordinarily difficult to reach. This piece is about a nice technique for making those dark corners easier to reach using either random and non-random testing. Although the method is one that I have used before, I hadn’t really thought about it explicitly until a conversation with Colin Runciman the other day brought it out into the open.

The issue is the small scope hypothesis, which states that most bugs can be triggered using small test cases. Here we’re talking about a class of bug that falls outside of this hypothesis: these bugs are triggered when some implementation limit is exceeded. For example, if we have a bounded buffer with length 10,000, and this buffer contains a bug in its handling of the buffer-full case, then it is clear that no test case can trigger the bug unless it results in more than 10,000 insertions into the buffer. Similarly, a bug in an operating system’s swapping code will not be hit until we put memory pressure on the machine — this will be difficult if we’re testing on a machine with lots of RAM. A bug in a compiler’s register spill logic will not be hit until we run out of registers, which may happen only rarely on a RISC with a large register file.

The testing technique, then, is simply to recognize capacities in the system under test and to artificially reduce them in order to make capacity bugs much easier to trigger. I think that probalby every accomplished programmer has used this technique, perhaps without even thinking about it: a simple example is testing the code for an auto-resizing hash table by making its initial capacity very small. This is just good sense.

Of course, while some capacities appear as explicit constants in the code that can be easily changed, others, such as the number of registers available to a compiler, will tend to be encoded in a more implicit fashion, and will therefore be more difficult to find and change. Also, the fact that a constant is hard-coded into a program doesn’t mean this constant can be freely changed: in some cases the constant is fake (how many programs that reference the CHAR\_BIT constant can really tolerate values other than 8?) and in others the program is only designed to operate for a certain range of choices. To continue with the compiler example, the calling convention (and other factors) will almost certainly cause failures when the number of registers falls below a certain number. Similarly, a pipe or a thread pool can easily cause the system to deadlock when it is instantiated with a capacity that is too small. These things are design constraints, not the implementation errors that we are looking for.

In summary, if you have a system that offers a choice of capacity for an internal resource, it is often very useful to reduce this capacity for testing purposes, even if this results in poor throughput. This effect is particularly pronounced during fuzzing, where the random walks through the state space that are induced by random test cases will tend to hover around mean values with high probability.
