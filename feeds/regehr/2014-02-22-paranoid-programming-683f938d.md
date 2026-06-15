---
title: Paranoid Programming
url: https://blog.regehr.org/archives/1106
published: "2014-02-22T05:02:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=1106
---

# Paranoid Programming

A lot of software defects stem from developers who lack a proper sense of paranoia. While paranoia can of course be taken to idiotic extremes, a healthy respect for Murphy’s Law is useful in programming. This post is a list of useful paranoid techniques that I hope to use as the basis for a lecture on paranoid programming for my “solid code” class — please leave a note if you have a favorite paranoid technique that I’ve missed.

- using plenty of [assertions](https://blog.regehr.org/archives/1091)

- validating and sanitizing inputs

- checking for all possible errors returned by library functions, not just likely ones

- erring on the side of caution when corrupted state is detected, for example by halting or restarting part or all of the system instead of attempting to repair and recover

- checksumming RAM and ROM contents in embedded systems

- minimizing the scope from which identifiers are visible

- using and paying attention to compiler warnings and static analysis results

- using and paying attention to the output of dynamic tools like Valgrind and Clang’s UBSan

- using (to whatever extent this is feasible) formal methods tools like Frama-C to verify tricky, important functions

- writing fuzzers even for software that shouldn’t need to be fuzzed

- leveraging the type system to get better guarantees, for example by avoiding unnecessary type casts and by putting some kinds of integer values into single-members structs when writing C

- for critical loops, writing down the loop variant and invariant

- if possible, building the code using different compilers and testing the resulting executables against each other

- using timeouts, retry loops, and watchdog timers when appropriate

- aggressively keeping functions simple

- aggressively avoiding mutable global state

- not using concurrency unless absolutely necessary

- taking advantage of whatever language support is available for controlling mutable state: const, pure functions, ownership types, etc.

- giving subsystems the least privileges they need to do their jobs, for example using [seccomp](http://lwn.net/Articles/332974/)

- testing, testing, testing, and coverage

- being aware of current best practices for things like choice of libraries

- tracking down transient failures rather than ignoring them

- disabling interrupts inside of interrupt handlers, even when timing arguments support the non-existence of nested interrupt handlers

- getting people who didn’t write the code to read the code

- conservative allocation of stack memory in environments where stacks are a constrained resource

- unit-testing library functions, particularly mathematical ones, instead of trusting that they are correct for all inputs

As a random example, several commenters (and [Bruce Dawson](http://randomascii.wordpress.com/) in email) took issue with the [volatile-qualified lock variable in this post](https://blog.regehr.org/archives/1097); my view is that this qualifier probably does no harm and might do some good by encouraging the compiler to be careful in how it touches the lock variable. This seems to me to be a reasonable application of paranoia; obviously others disagree.

The other day someone asked me how paranoid programming differs from regular old defensive programming. I don’t necessarily have a great answer but (1) the top 10 google hits for “defensive programming” seemed to be missing many of the techniques I’ve listed here and (2) “defensive” doesn’t quite capture the depth of my mistrust of computer programs. Just because you’re paranoid doesn’t mean they’re not really out to get you, and make no mistake: the computer is out to get you (as are the people on the other side of the network).
