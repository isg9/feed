---
title: three approaches to heap sizing — wingolog
url: https://wingolog.org/archives/2023/01/27/three-approaches-to-heap-sizing
published: "2023-01-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/01/27/three-approaches-to-heap-sizing
---

# three approaches to heap sizing — wingolog

## [three approaches to heap sizing](/archives/2023/01/27/three-approaches-to-heap-sizing)

27 January 2023 9:45 PM

- [garbage collection](/tags/garbage%20collection)
- [heap sizing](/tags/heap%20sizing)
- [membalancer](/tags/membalancer)
- [gc](/tags/gc)
- [igalia](/tags/igalia)
- [guile](/tags/guile)
- [science](/tags/science)
- [knobs](/tags/knobs)

How much memory should a program get? Tonight, a quick note on sizing for garbage-collected heaps. There are a few possible answers, depending on what your goals are for the system.

### you: doctor science

Sometimes you build a system and you want to study it: to identify its principal components and see how they work together, or to isolate the effect of altering a single component. In that case, what you want is a fixed heap size. You run your program a few times and determine a heap size that is sufficient for your problem, and then in future run the program with that new fixed heap size. This allows you to concentrate on the other components of the system.

A good approach to choosing the fixed heap size for a program is to determine the minimum heap size a program can have by bisection, then multiplying that size by a constant factor. Garbage collection is a space/time tradeoff: the factor you choose represents a point on the space/time tradeoff curve. I would choose 1.5 in general, but this is arbitrary; I’d go more with 3 or even 5 if memory isn’t scarce and I’m really optimizing for throughput.

Note that a fixed-size heap is not *generally* what you want. It’s not good user experience for running `./foo` at the command line, for example. The reason for this is that program memory use is usually a function of the program’s input, and only in some cases do you know what the input might look like, and until you run the program you don’t know what the exact effect of input on memory is. Still, if you have a team of operations people that knows what input patterns look like and has experience with a GC-using server-side process, fixed heap sizes could be a good solution there too.

### you: average josé/fina

On the other end of the spectrum is the average user. You just want to run your program. The program should have the memory it needs! Not too much of course; that would be wasteful. Not too little either; I can tell you, my house is less than 100m², and I spend way too much time shuffling things from one surface to another. If I had more space I could avoid this wasted effort, and in a similar way, you don’t want to be too stingy with a program’s heap. Do the right thing!

Of course, you probably have multiple programs running on a system that are making similar heap sizing choices at the same time, and the relative needs and importances of these programs could change over time, for example as you switch tabs in a web browser, so the right thing really refers to overall system performance, whereas what you are controlling is just one process’ heap size; what is the Right Thing, anyway?

[My corner of the GC discourse
agrees](https://twitter.com/andywingo/status/1618521467230248960) that [something like the right solution was outlined by Kirisame, Shenoy, and
Panchekha in a 2022 OOPSLA
paper](https://2022.splashcon.org/details/splash-2022-oopsla/60/Optimal-Heap-Limits-for-Reducing-Browser-Memory-Use), in which the optimum heap size depends on the allocation rate and the gc cost for a process, which you measure on an ongoing basis. Interestingly, their formulation of heap size calculation can be made by each process without coordination, but results in a whole-system optimum.

There are some details but you can imagine some instinctive results: for example, when a program stops allocating because it’s waiting for some external event like user input, it doesn’t need so much memory, so it can start shrinking its heap. After all, it might be quite a while before the program has new input. If the program starts allocating again, perhaps because there is new input, it can grow its heap rapidly, and might then shrink again later. The mechanism by which this happens is pleasantly simple, and I salute (again!) the authors for identifying the practical benefits that an abstract model brings to the problem domain.

### you: a damaged, suspicious individual

Hoo, friends– I don’t know. I’ve seen some things. Not to exaggerate, I like to think I’m a well-balanced sort of fellow, but there’s some suspicion too, right? So when I imagine a background thread determining that my web server hasn’t gotten so much action in the last 100ms and that really what it needs to be doing is shrinking its heap, kicking off additional work to mark-compact it or whatever, when the whole point of the virtual machine is to run that web server and not much else, only to have to probably give it more heap 50ms later, I– well, again, I exaggerate. The MemBalancer paper has a heartbeat period of 1 Hz and a smoothing function for the heap size, but it just smells like danger. Do I need danger? I mean, maybe? Probably in most cases? But maybe it would be better to avoid danger if I can. Heap growth is usually both necessary and cheap when it happens, but shrinkage is never necessary and is sometimes expensive because you have to shuffle around data.

So, I think there is probably a case for a third mode: not *fixed*, not *adaptive* like the MemBalancer approach, but just *growable*: grow the heap when and if its size is less than a configurable multiplier (e.g. 1.5) of live data. Never shrink the heap. If you ever notice that a process is taking too much memory, manually kill it and start over, or whatever. Default to adaptive, of course, but when you start to troubleshoot a high GC overhead in a long-lived proess, perhaps switch to growable to see its effect.

### unavoidable badness

There is some heuristic badness that one cannot avoid: even with the adaptive MemBalancer approach, you have to choose a point on the space/time tradeoff curve. Regardless of what you do, your system will grow a hairy nest of knobs and dials, and if your system is successful there will be a lively aftermarket industry of tuning articles: “Are you experiencing poor object transit? One knob you must know”; “Four knobs to heaven”; “It’s raining knobs”; “GC engineers DO NOT want you to grab this knob!!”; etc. (I hope that my British readers are enjoying this.)

These ad-hoc heuristics are just part of the domain. What I want to say though is that having a general framework for how you approach heap sizing can limit knob profusion, and can help you organize what you have into a structure of sorts.

At least, this is what I tell myself; inshallah. Now I have told you too. Until next time, happy hacking!

## related articles

- [whippet progress update: feature-complete!](/archives/2024/09/18/whippet-progress-update-feature-complete)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)

Comments are closed.
