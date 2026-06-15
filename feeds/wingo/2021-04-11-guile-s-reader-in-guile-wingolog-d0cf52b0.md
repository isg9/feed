---
title: guile's reader, in guile — wingolog
url: https://wingolog.org/archives/2021/04/11/guiles-reader-in-guile
published: "2021-04-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2021/04/11/guiles-reader-in-guile
---

# guile's reader, in guile — wingolog

## [guile's reader, in guile](/archives/2021/04/11/guiles-reader-in-guile)

11 April 2021 7:51 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [bootstrapping](/tags/bootstrapping)
- [read](/tags/read)
- [parsing](/tags/parsing)

Good evening! A brief(ish?) note today about some Guile nargery.

**the arc of history**

Like many language implementations that started life when you could turn on the radio and expect to hear Def Leppard, Guile has a bottom half and a top half. The bottom half is written in C and exposes a shared library and an executable, and the top half is written in the language itself (Scheme, in the case of Guile) and somehow loaded by the C code when the language implementation starts.

Since 2010 or so we have been working at replacing bits written in C with bits written in Scheme. Last week's missive was about [replacing the implementation of `dynamic-link`](https://wingolog.org/archives/2021/04/08/sign-of-the-times) from using the `libltdl` library to using Scheme on top of a low-level `dlopen` wrapper. I've written about [rewriting `eval` in Scheme](https://wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty), and more recently about how [the road to getting the performance of C implementations in Scheme has been sometimes long](https://wingolog.org/archives/2020/02/07/lessons-learned-from-guile-the-ancient-spry).

These rewrites have a quixotic aspect to them. I feel something in my gut about rightness and wrongness and I know at a base level that moving from C to Scheme is the right thing. Much of it is completely irrational and can be out of place in a lot of contexts -- like if you have a task to get done for a customer, you need to sit and think about minimal steps from here to the goal and the gut doesn't have much of a role to play in how you get there. But it's nice to have a project where you can do a thing in the way you'd like, and if it takes 10 years, that's fine.

But besides the ineffable motivations, there are concrete advantages to rewriting something in Scheme. I find Scheme code to be more maintainable, yes, and [more secure relative to the common pitfalls of C](https://noncombatant.org/2021/04/09/prioritizing-memory-safety-migrations/), obviously. It decreases the amount of work I will have when one day I rewrite Guile's garbage collector. But also, Scheme code gets things that C can't have: tail calls, resumable delimited continuations, run-time instrumentation, and so on.

Taking delimited continuations as an example, five years ago or so I wrote a [lightweight concurrency facility for Guile](https://wingolog.org/archives/2017/06/29/a-new-concurrent-ml), modelled on Parallel Concurrent ML. It lets millions of fibers to exist on a system. When a fiber would need to block on an I/O operation (read or write), instead it suspends its continuation, and arranges to restart it when the operation becomes possible.

A lot had to change in Guile for this to become a reality. Firstly, [delimited continuations](https://wingolog.org/archives/2010/02/26/guile-and-delimited-continuations) themselves. Later, a complete [rewrite of the top half of the ports facility in Scheme](https://www.gnu.org/software/guile/docs/master/guile.html/Non_002dBlocking-I_002fO.html), to allow port operations to suspend and resume. Many of the barriers to resumable fibers were removed, but the [Fibers manual](https://github.com/wingo/fibers/wiki/Manual#Blocking) still names quite a few.

**Scheme `read`, in Scheme**

Which brings us to today's note: I just rewrote Guile's reader in Scheme too! The reader is the bit that takes a stream of characters and parses it into S-expressions. It was in C, and now is in Scheme.

One of the primary motivators for this was to allow `read` to be suspendable. With this change, read-eval-print loops are now implementable on fibers.

Another motivation was to finally fix a bug in which Guile couldn't record source locations for some kinds of datums. It used to be that Guile would use a weak-key hash table to associate datums returned from `read` with source locations. But this only works for fresh values, not for immediate values like small integers or characters, nor does it work for globally unique non-immediates like keywords and symbols. So for these, we just wouldn't have any source locations.

A robust solution to that problem is to return annotated objects rather than using a side table. Since Scheme's macro expander is already set to work with annotated objects ( [syntax objects](https://www.gnu.org/software/guile/manual/html_node/Syntax-Transformer-Helpers.html)), a new `read-syntax` interface would do us a treat.

With `read` in C, this was hard to do. But with `read` in Scheme, it was no problem to implement. Adapting the expander to expect source locations inside syntax objects was a bit fiddly, though, and the resulting increase in source location information makes the output files bigger by a few percent -- due somewhat to the increased size of the `.debug_lines` DWARF data, but also due to serialized source locations for syntax objects in macros.

Speed-wise, switching to `read` in Scheme is a regression, currently. The old reader could parse around 15 or 16 megabytes per second when recording source locations on this laptop, or around 22 or 23 MB/s with source locations off. The new one parses more like 10.5 MB/s, or 13.5 MB/s with positions off, when in the old mode where it uses a weak-key side table to record source locations. The new `read-syntax` runs at around 12 MB/s. We'll be noodling at these in the coming months, but unlike when the original reader was written, at least now the reader is mainly used only at compile time. (It still has a role when reading s-expressions as data, so there is still a reason to make it fast.)

As is the case with [`eval`](https://wingolog.org/archives/2016/01/11/the-half-strap-self-hosting-and-guile), we still have a C version of the reader available for bootstrapping purposes, before the Scheme version is loaded. Happily, with this rewrite I was able to remove all of the cruft from the C reader related to non-default lexical syntax, which simplifies maintenance going forward.

An interesting aspect of attempting to make a bug-for-bug rewrite is that you find bugs and unexpected behavior. For example, it turns out that since the dawn of time, Guile always read `#t` and `#f` without requiring a terminating delimiter, so reading "(#t1)" would result in the list `(#t 1)`. Weird, right? Weirder still, when the `#true` and `#false` aliases were added to the language, Guile decided to support them by default, but in an oddly backwards-compatible way... so "(#false1)" reads as `(#f 1)` but "(#falsa1)" reads as `(#f alsa1)`. Quite a few more things like that.

All in all it would seem to be a successful rewrite, introducing no new behavior, even producing the same errors. However, this is not the case for backtraces, which can expose the guts of `read` in cases where that previously wouldn't happen because the C stack was opaque to Scheme. Probably we will simply need to add more sensible error handling around callers to `read`, as a backtrace isn't a good user-facing error anyway.

OK enough rambling for this evening. Happy hacking to all and to all a good night!

## related articles

- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

### 2 responses

1. tomás zerolo says:[12 April 2021 10:01 AM](#0dc14295021a0526106e551d84156f1e05e2bbf8)

   "With this change, read-eval-print loops are now implementable on fibers"

   Yay!

   And thanks again for making difficult stuff so readable.

2. Luís says:[17 April 2021 11:48 PM](#84c9f4da763b7074753eef0bba8d117a9dc2b9b7)

   Cool stuff. Out of curiosity, what made you keep the bugs, if you're in do-the-right-thing mode?

Comments are closed.
