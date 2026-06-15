---
title: musl 1.1.5 with full C11 library&nbsp;support
url: https://gustedt.wordpress.com/2014/10/14/musl-1-1-5-with-full-c11-library-support/
published: "2014-10-14T20:52:25Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=2191
---

# musl 1.1.5 with full C11 library&nbsp;support

Today, Rich Felker has published the next release of musl the lightweight, standard conforming C library. He says:

> This release adds full library-level support for C11, including the
>
> C11 threads API, and features major performance and correctness
>
> improvements to the implementations of thread synchronization
>
> primitives, especially condition variables. Several serious bugs have
>
> been fixed, including a failure to null-terminate certain unexpected
>
> DNS replies, use of uninitialized memory in caller-provided thread
>
> stacks, writes past the end of the buffer when `swab()` is called
>
> with an odd length, and missing memory barriers in several places.
>
> Many other minor bugs have also been fixed.

It can be found at the obvious [place.](http://www.musl-libc.org/releases/musl-1.1.5.tar.gz "place")

I took part in that experience, in particular for the implementation of the C threads API, and for revising some of the POSIX lock structures. First of all, it has been really fun and a nice experience. Musl is a friendly and competent community, Rich is a very patient sparring partner to throw crazy ideas at. Thanks Rich!

This C threads implementation is mostly based on the existing POSIX threads implementation in musl and so is in some points an interpretation of the C standard, were it lacks specification. There are some weak points in that specification, I already talked about some, and during the implementation process we found some new ones. Fortunately nothing that is a show stopper, so musl seems to be the first open source C library out there, that has full support for C11. This is complemented by the newer versions of gcc and clang that seem to support the compiler part of C11. Also, they support atomics in all their glory.

So this gives you **full support for C11** in the **open**.

Perhaps people still underestimate the usefulness of C threads:

- They have simpler interfaces, no “attributes” for mutexes or condition variables and alike.
- They come with well-defined atomic data types and operations. All modern architectures support this feature in one form or another, and up to now we only had access to that through awful assembler hacks.
- They allow for a different memory model without stack sharing between threads.
- They can also be implemented in non-POSIX operating systems, such as in the formerly dominant desktop OS, or on embedded systems, maybe even on “bare metal”.
- They have no model for thread cancellation.

Especially the first two make it a very attractive thread API for simple parallel programming experiences, including for teaching.

The lack of cancellation is an important property, too. We spent a lot of time this summer discussing the interaction between mutexes, condition variables and cancellation, and believe me taking out cancellation from the picture eases the argumentation (and thus implementation) *a lot*. To state it bluntly, I found the POSIX model insane: it has three different flows of events that interact with each other in unexpected ways.

1. The first level is relatively simple, mutexes. Operations on mutexes give a strictly synchronized flow of events between all threads that use the same mutex. Good.
2. Condition variables darken the picture a bit. We all know that to go into a wait on a condition variable a thread has to hold the mutex in question. So waiting synchronizes well with the other operations on the mutex. Unfortunately, to wake up other threads by `signal` or `broadcast` we are *not* required to hold the mutex. So, for each condition variable there is another flow of events *for each thread* that uses it, that is not synchronized with the events on the mutex.
3. Add cancellation to that picture to obtain a real mess. We have a third type of events that are triggered by a thread that might have nothing to do with the condition variable or the mutex. A nightmare, to keep track of a “happened before” relation, to keep data structures consistent, to implement at all. All this for a feature that is rarely used and even less understood by programmers.

So, overall my hope is that C threads may gain more momentum and that a thread model that is a bit saner, more user-friendly and that reflects the properties of modern hardware will prevail.
