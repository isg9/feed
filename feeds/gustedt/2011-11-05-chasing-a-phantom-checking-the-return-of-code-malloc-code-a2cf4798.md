---
title: 'chasing a phantom: checking the return of <code>malloc</code>'
url: https://gustedt.wordpress.com/2011/11/05/chasing-a-phantom-checking-the-return-of-malloc/
published: "2011-11-05T22:46:28Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1141
---

# chasing a phantom: checking the return of <code>malloc</code>

Often you see or hear as one of the first rules that are taught about the use of `malloc` (and derivatives) that you’d have to check the return value, to see if it is `0` and thus to know whether it failed. Although there are situations in which `malloc` may fail and that this check makes sense, doing so gives you mostly false security. In most situations where this might fail you are in trouble for quite a while and the user of the program (if any) will most probably have aborted the execution since long.

Don’t understand me wrong, I don’t say you shouldn’t ever check the return of `malloc`, I just will try to show you that there are many other things that have to be considered before that, and that to my opinion have much more importance. They *are important* on systems that have very restrictive possibilities for memory allocation. Usually these are so-called *freestanding* environments: embedded devices, space rockets, Linux kernel, or other very specialized stuff. Programming on multi-commodity architectures ( *hosted* environment)is quite different from that.

`malloc` can return `0` under two different circumstances:

1. The object size that was to be allocated was specified to be zero
2. The memory system is saturated and an object of that size could not be allocated

## Zero size allocations

The first condition is not imposed by the standard. When called with an argument of `0` the standard allows `malloc` to return a sort of unique address, which is not a null pointer and which you don’t have the right to access, nevertheless. It just imposes that such an address must also be a valid argument for `free`. So

> **Checking the return of `malloc` against `0` doesn’t protect you from allocation of zero-sized objects.**

Always check the argument(s) to malloc and Co. or make sure by other means that you will not pass `0` to `malloc`. The way to avoid that is to avoid `malloc` in the first place. Use local ( `auto` or `register`) variables whenever you may. If you are allocating an object of a primitive data type ( `int`, `double`, `void*` …) with `malloc` (and conceptually this is not an array of length `1`) you are most probably on the wrong track. Rethink your design.

The second easiest way to that is to never use explicit arithmetic as an argument to `malloc`. Always use a `sizeof` expression. In the simplest case this is just the variable that you are dealing with

```
strongNode * eff = malloc(sizeof *eff);

```

such a `sizeof` expression will never be zero. As an additional feature the type of `eff` is only specified once, if you later change it to `weakNode`, say, the rest will still work.

Even for vector allocations you don’t need explicit arithmetic. Just do something like

```
double * geh = malloc(sizeof(double[en]));

```

Technically, this uses the size of a variable length array, but this shouldn’t bother you. This is just a readable variant of telling `malloc` to allocate the space for the array type that you have in mind.

In P99 there is a convenience macro [`P99_MALLOC`](http://p99.gforge.inira.fr/p99-html/group__preprocessor__allocation.html "preprocessor allocation") that condenses the `malloc` and the `sizeof` into one.

```
strongNode * eff = P99_MALLOC(*eff);
double * geh = P99_MALLOC(double[en]);

```

## Memory subsystem failure

The second error condition (memory exhausted) on the other hand is quite rare in *hosted* environments. Usually you are in trouble long before you hassle with this kind of error. On a hosted environment `malloc` doesn’t allocate physical memory (this or that memory bank of your computer) but allocates memory in a virtual address space. This virtual address space is an abstraction that is provided by the OS that lets your process live in a sandbox where it is relatively undisturbed by other processes, and where the system juggles data between different forms of storage facilities (cache memory of different levels, RAM, “disk” such as hard-disk or SSD).

This world of the virtual address space is a well maintained *fiction*, that helps us to survive the daily challenge of programming. Just like *[Zaphod Beeblebrox](http://en.wikipedia.org/wiki/Zaphod_Beeblebrox "Zaphod Beeblebrox")* only was able to survive the *Total Perspective Vortex* because he affronted it in a specially constructed, parallel universe. But just as for Zaphod, there might be problems in the real universe that you will not trick by just ignoring them.

As said virtual address space is usually backed by different types of devices, that have quite different properties, in particular quite different bandwidth ( *what throughput do I get*) and latency ( *how fast do I get an answer*). For practical purposes this means:

> **A system that is out of memory is unresponsive loooooong time before `malloc` will fail.**

To that difficulty of a hanging systems comes an extra property that a modern system (notably Linux) might have, *overcommitment*. This is when virtual address space isn’t even backed up by some physical device. The system happily assigns addresses to your process that then are returned by `malloc` and it only faults when you *access* a page for which there is no physical correspondence. But when programming “normally” you will always get hit by the unresponsiveness of the system. Exhausting the 248 virtual addresses of a modern system and having `malloc` fail for that reason would be a sportive challenge.

To my experience, in 99% of the cases a program execution that runs out of memory is the result of a *serious bug*. Only the other 1% is due to a platform/problem/user combination reaching the limits. To avoid the first 99% you can use several things

- Review your code and have it reviewed by someone else. In particular assure yourself that all allocations are freed somewhere. *All*.

- Use an allocation checking tool like [valgrind](http://valgrind.org/ "valgrind home page"). Test your program under different scenarios and don’t stop until valgrind finds no leak at all.

Only then it makes sense of asking your self the question, “ *what if I run out of memory?*“. And now you may see if checking for the return of `malloc` at a particular place of your code or perhaps the redesign of your data structures is the right answer.

## Coding Style

Another disadvantage of the “ *check the return of `malloc`*” idiom is code readability. People tend to decorate their code with complicated captures of that error condition that span several lines, doing `perror` and stuff like that. As already [The Elders](http://www.kernel.org/doc/Documentation/CodingStyle "kernel coding style") said (citation taken a bit out of context)

> **… the supply of new-lines on your screen is not a renewable resource …**

there might be a trade-off between explicit error checking and readability of your code. If error checking conditions dominate your code and distract from the essential control flow you have a design problem. Wrap your error checking in a function or macro to avoid that.

The easiest, to my taste, would be something like `memset(malloc(n), 0, 1)`. This just writes a `0` in the first byte and crashes nicely if `malloc` had an error or `n` was `0` to begin with.

A good coding style is also to always initialize variables. In particular this is crucial for pointer variables:

- Situations where the compiler will not be able to optimize an initialization if it is overwritten by an assignment are rare. Only optimize for that case it a profiler tells you that *there* is a performance bottleneck.

- These situations are most often even easy to avoid in the first place. C99 allows you to declare a variable at its first use. Such a first use *must* be an assignment (otherwise you have undefined behavior) so why not just declare it there, where it pops into live, and initialized it directly.

- [Initialize `struct` variables](http://gustedt.wordpress.com/2010/06/19/consistent-initialization-of-static-and-auto-variables/) with initializer expressions enclosed in `{}`. This guarantees you that forgotten pointer members are initialized to `0`.
- Write a [“init” function](http://gustedt.wordpress.com/2010/06/22/initialization-of-dynamically-allocated-struct/ "initialization of dynamically allocated struct") for every `struct` type that is allocated through `malloc`. As an extra bonus you then get [`P99_NEW`](http://p99.gforge.inira.fr/p99-html/group__preprocessor__allocation.html "preprocessor allocation") that lets you easily allocate an initialize an object in one go.
