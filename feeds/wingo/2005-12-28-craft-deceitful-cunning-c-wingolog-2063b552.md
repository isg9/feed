---
title: Craft; deceitful cunning; &c. — wingolog
url: https://wingolog.org/archives/2005/12/28/craft-deceitful-cunning-c
published: "2005-12-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/12/28/craft-deceitful-cunning-c
---

# Craft; deceitful cunning; &c. — wingolog

## [Craft; deceitful cunning; &c.](/archives/2005/12/28/craft-deceitful-cunning-c)

29 December 2005 0:43 AM

- [scheme](/tags/scheme)

It's been a long time since I rapped at ya played with [Guile](http://www.gnu.org/software/guile/guile.html), but holiday has given me the opportunity to get back to it.

Guile is an implementation of Scheme that's designed to integrate well with existing C programs and libraries. Most of the primitives that the interpreter offers are implemented in C, which means that you can access them from either language. For example, `(hash-set! h key val)` in Scheme is equivalent to `scm_hash_set_x (h, key, val)` in C, for suitable values of `h`, `key`, and `val`.

**Frames**

The parity between languages extends to exceptions. Because the semantics of Scheme specify that an exception travels up the control stack, and Guile has to integrate well with C programs that already have their own control stack, Guile implements exceptions using longjmp. Guile annotates the C stack with debugging information, such that there is one "frame" on the C stack for every block on the Scheme stack.

So, if you call `scm_hash_set_x (h, k, val)` in C, but `h` is not a hash table, the function will never return. Instead, Guile starts crawling back up the C stack looking for Guile frames, stopping when there is an exception handler. In this way it is similar to the JVM or the CLR, and unlike e.g. Python or the Unix system call interface, which return special values and rely on the programmer to check errno or PyErr\_foo.

Guile 1.7 adds explicit support for frames from C. For example, consider the following block of code, which comes from Guile's NEWS file:

```
  void
  foo ()
  {
    char *mem;

    scm_frame_begin (0);

    mem = scm_malloc (100);
    scm_frame_unwind_handler (free, mem, SCM_F_WIND_EXPLICITLY);

    /* MEM would leak if BAR throws an error.
       SCM_FRAME_UNWIND_HANDLER frees it nevertheless.
     */

    bar ();

    scm_frame_end ();

    /* Because of SCM_F_WIND_EXPLICITLY, MEM will be freed by
       SCM_FRAME_END as well.
    */
  }

```

Guile inherits the SCM namespace from [SCM](http://www.swiss.ai.mit.edu/~jaffer/SCM.html), Aubrey Jaffer's Scheme interpreter that Guile forked from about 10 years ago. For this particular case, Guile defines `scm_frame_free (mem)` as being a shorthand for this invocation of `scm_frame_unwind_handler`. There are similar shorthands for setting values of [fluids](http://www.gnu.org/software/guile/docs/guile-ref/Fluids.html) and for setting the current input/output/error ports.

Additionally, to properly support continuations, Guile supports multiple non-local reentries from points that were exited via call-with-current-continuation. That, however, is a story for another day.

**Threads**

Also new with Guile 1.7 is full support for OS-level threads. As evaluation state is captured on the C stack instead of in a global interpreter state, this is easier than in most languages -- since each thread has its own stack, the threads can run concurrently (unlike e.g. Python). The only part of Guile that needed to change was the garbage collector, and those changes are a bit boring unless you're a language nerd.

More interesting are the possibilities offered by threads to the programmer. There are "futures", which are similar to [promises](http://www.swiss.ai.mit.edu/~jaffer/r5rs_8.html#IDX414) except that they begin executing immediately in another thread. There is `letpar`, like the `let` binding construct except that it spawns threads to calculate the binding values, and `par-map`, similarly like `map` but parallelized.

There are also nestable `parallelize` and `serialize` forms, which can be used to control the threading behavior of particular blocks of code, for example to serialize access to a thread-unsafe C library. And of course, there is the normal range of mutexen, semaphores, and condition variables. Best of all, Guile is guaranteed not to crash under any circumstances. (Not saying that it won't crash, of course, but that if it does it's a bug in Guile.)

The culture of Scheme places a strong emphasis on the functional programming style. Scheme programmers learn early on how to identify shared state, which is a large part of understanding concurrency. This makes a threads-supporting Scheme like Guile an ideal language in which to learn about multiprocessing.

**Other Bling**

Guile 1.7 now uses [gmp](http://www.gnu.org/software/gmp/) for its arithmetic, obsoleting the old hand-rolled implementation of bignums. Using gmp also allows for infinity, NaN, and signed zero numeric values.

Uniform vectors -- N-dimensional, potentially sparse arrays of one data type, for example 16-bit integers -- have long been supported by Guile. Now their implementation conforms to the [srfi-4 standard](http://srfi.schemers.org/srfi-4/srfi-4.html), and is always built.

Dividing 12 by 18 will now yield the exact rational number 2/3, completing the numeric tower of types. Oh yes. I'm picking on Python a bit, but it's all in good fun so I'll continue: they just changed from 12/18==0 to 12/18==0.66666666666666663, which is still inaccurate. (And yes, I do get that 3 on the end on this machine. Odd.)

Finally, functions to convert between SCM values and C types were made much more uniform. No more cryptic macros in new code (SCM\_MAKINUM anyone?).

**Subjectivist**

Hacking in Guile makes me happy. I see trees of code folding and rearranging themselves, tail calls jumping around, and at the end and beginning there is the blinking cursor at the `guile> ` prompt, waiting for me. Definitely a resolution for 2006 to spend more time scheming.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

Comments are closed.
