---
title: 'correct or inotify: pick one — wingolog'
url: https://wingolog.org/archives/2018/05/21/correct-or-inotify-pick-one
published: "2018-05-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/05/21/correct-or-inotify-pick-one
---

# correct or inotify: pick one — wingolog

## [correct or inotify: pick one](/archives/2018/05/21/correct-or-inotify-pick-one)

21 May 2018 2:29 PM

- [inotify](/tags/inotify)
- [concurrency](/tags/concurrency)
- [linux](/tags/linux)
- [unix](/tags/unix)
- [igalia](/tags/igalia)

Let's say you decide that you'd like to see what some other processes on your system are doing to a subtree of the file system. You don't want to have to change how those processes work -- you just want to see what files those processes create and delete.

One approach would be to just scan the file-system tree periodically, enumerating its contents. But when the file system tree is large and the change rate is low, that's not an optimal thing to do.

Fortunately, Linux provides an API to allow a process to receive notifications on file-system change events, called `inotify`. So you open up the [`inotify(7)`](http://man7.org/linux/man-pages/man7/inotify.7.html) manual page, and are greeted with this:

> With careful programming, an application can use inotify to efficiently monitor and cache the state of a set of filesystem objects. However, robust applications should allow for the fact that bugs in the monitoring logic or races of the kind described below may leave the cache inconsistent with the filesystem state. It is probably wise to do some consistency checking, and rebuild the cache when inconsistencies are detected.

It's not exactly reassuring is it? I mean, "you had one job" and all.

Reading down a bit farther, I thought that with some "careful programming", I could get by. After a day of trying, I am now certain that it is impossible to build a correct recursive directory monitor with `inotify`, and I am not even sure that "good enough" solutions exist.

**pitfall the first: buffer overflow**

Fundamentally, `inotify` races the monitoring process with all other processes on the system. Events are delivered to the monitoring process via a fixed-size buffer that can overflow, and the monitoring process provides no back-pressure on the system's rate of filesystem modifications. With `inotify`, you have to be ready to lose events.

This I think is probably the easiest limitation to work around. The kernel can let you know when the buffer overflows, and you can tweak the buffer size. Still, it's a first indication that perfect is not possible.

**pitfall the second: now you see it, now you don't**

This one is the real kicker. Say you get an event that says that a file "frenemies.txt" has been created in the directory "/contacts/". You go to open the file -- but is it still there? By the time you get around to looking for it, it could have been deleted, or renamed, or maybe even created again or replaced! This is a [TOCTTOU](https://en.wikipedia.org/wiki/Time_of_check_to_time_of_use) race, built-in to the `inotify` API. It is literally impossible to use `inotify` without this class of error.

The canonical solution to this kind of issue in the kernel is to use file descriptors instead. Instead of or possibly in addition to getting a name with the file change event, you get a descriptor to a (possibly-unlinked) open file, which you would then be responsible for closing. But that's not what `inotify` does. Oh well!

**pitfall the third: race conditions between inotify instances**

When you `inotify` a directory, you get change notifications for just that directory. If you want to get change notifications for subdirectories, you need to open more `inotify` instances and `poll` on them all. However now you have *N2* problems: as `poll` and the like return an unordered set of readable file descriptors, each with their own ordering, you no longer have access to a linear order in which changes occurred.

It is *impossible* to build a recursive directory watcher that definitively says "ok, first `/contacts/frenemies.txt` was created, then `/contacts` was renamed to `/peeps`, ..." because you have no ordering between the different watches. You don't know that there was *ever* even a time that `/contacts/frenemies.txt` was an accessible file name; it could have been only ever openable as `/peeps/frenemies.txt`.

Of course, this is the most basic ordering problem. If you are building a monitoring tool that actually wants to open files -- good luck bubster! It literally cannot be correct. (It might work well enough, of course.)

**reflections**

As far as I am aware, `inotify` came out to address the needs of desktop search tools like the belated [Beagle](https://en.wikipedia.org/wiki/Beagle_(software)) (11/10 good pupper just trying to get his pup on). Especially in the days of spinning metal, grovelling over the whole hard-drive was a real non-starter, especially if the search database should to be up-to-date.

But after looking into `inotify`, I start to see why someone at Google said that desktop search was in some ways harder than web search -- I mean we all struggle to find files on our own machines, even now, 15 years after the whole dnotify/inotify thing started. Part of it is that the given the choice between supporting reliable, fool-proof file system indexes on the one hand, and overclocking the IOPS benchmarks on the other, the kernel gave us `inotify`. I understand it, but `inotify` still sucks.

I dunno about you all but whenever I've had to document such an egregious uncorrectable failure mode as any of the ones in the `inotify` manual, I have rewritten the software instead. In that spirit, I hope that some day we shall send `inotify` to the pet cemetery, to rest in peace beside Beagle.

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [missing the point of webassembly](/archives/2024/01/08/missing-the-point-of-webassembly)
- [the last 5 years of V8's garbage collector](/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector)
- [on "correct and efficient work-stealing for weak memory models"](/archives/2022/10/03/on-correct-and-efficient-work-stealing-for-weak-memory-models)
- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)
- [unintentional concurrency](/archives/2022/07/20/unintentional-concurrency)

### 5 responses

1. Anon says:[21 May 2018 3:39 PM](#8c9aa34f61a5d6f3942b565b23feebfcf821eec6)

   There is also fanotify https://lwn.net/Articles/339399/

   I'm not sure it solves any problem for you, but IIRC it was create to overcome problems with inotify

2. Jos says:[21 May 2018 7:09 PM](#a10440bc249bd57a3075a18728b6d2e3b34363ec)

   The state of watching file system events without needing priviliges is indeed lamentable. You've not even mentioned the insane amount of memory needed to keep inotify monitors on all directories: userspace is responsible for mapping the fd in the event back to the full path. So the client has to keep all directory paths in memory.

   An efficient kernel interface for this is really needed. Even with fast SSD, the overhead of doing find is still high.

   find ~ -printf '%T@\\t%p\\n'\|wc -l

   on a home dir with 1.5M files takes 35 seconds on my laptop with fast SSD.

3. Michael Catanzaro says:[21 May 2018 9:30 PM](#00dc931477f83be50ddfa897767a238a73767d97)

   Michael Kerrisk discusses many of these problems in https://lwn.net/Articles/605128/.

4. Helge Bahmann says:[24 May 2018 8:33 AM](#793db917d3c5b392f985b9c3b7d66c0a7f530592)

   I don't think inotify (or fanotify FWIW) was intended for the purposes you describe ("see what some other processes on your system are doing to a subtree of the file system"). It was intended to allow applications to catch up with the \*state\* of a fileset (or tree) in an "eventually consistent (!)" fashion. As such, it is not intended to (and cannot) replace active scanning/reading of filesystem state, but to cut down on the rate/scope of when/what to scan. Basically: "no event" -> just sleep. "any event" -> rescan such that we have covered everything potentially affected by last event (worst case whole tree).

   For activity trace with temporal ordering it is more efficient to trace the actors (=processes) instead of the objects (=files) because there are far fewer. No file-based notification API can give you that.

5. Marc says:[24 May 2018 2:33 PM](#36f6310de03a6b9d4b7ba12cfbedc8a18d3c98ba)

   Yeah I think you're confusing "inotify doesn't work" with "inotify doesn't scale" (to multiple directories, processes, etc.)

Comments are closed.
