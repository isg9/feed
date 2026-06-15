---
title: Semaphores and Mutexes aren&#8217;t so&nbsp;similar
url: https://gustedt.wordpress.com/2010/06/29/semaphores/
published: "2010-06-29T12:53:54Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=126
---

# Semaphores and Mutexes aren&#8217;t so&nbsp;similar

A common misconception I read about the two POSIX types `sem_t` and `pthread_mutex_t` are that that they’d be similar, almost interchangeable it seems. They have indeed *some* things in common:

- both are control structures that can be placed in memory that is shared between threads (or processes, we will concentrate on threads, here)
- both can be used to block a thread to wait for another one
- both have three different variants for such a wait function
  1. Unconditional, blocking wait, `sem_wait` and `pthread_mutex_lock`.
  2. Non-blocking wait, `sem_trywait` and `pthread_mutex_trylock`.
  3. Time dependent wait, `sem_timedwait` and `pthread_mutex_timedlock`.

But in other aspects they are quite different. The main difference, already on a conceptual level that I see is the following:

- pthread\_mutex\_t is thread oriented. If a mutex is hold by a particular thread it can only be unlocked by this particular thread alone.
- sem\_t is token oriented. If a thread is blocked on a semaphore, any active thread may post on the semaphore.

This already reserves certain surprises to the unaware, and each of these concepts as it own sets of pitfalls that have to be taken care of.

But then, the special specification for the mutex and semaphore *concepts* as they are given by POSIX adds a particular oddity that makes their use a bit tricky for starters:

- The semaphore calls can be interrupted at any time, e.g by IO that is delivered or if a process child terminates. In such a case `errno` is set to `EINTR`.
- A thread **A** that is blocked in `pthread_mutex_lock` can receive as much signals as it would. It will only return to the application once the thread **B** that holds the lock releases it. If **B** returns, crashes, idles for whatever reason, **A** will be blocked.

The reason for that is just that sem\_t is designed to be used in signal handlers, so it must be sensible to the fact that an interrupt arrived. `phtread_mutex_t` is better not used in interrupt handlers, unless you really know what you are doing.

In subsequent blogs, I will try to show the particular advantages or pitfalls of the two different interfaces

- [how to make `sem_t` interrupt safe](https://gustedt.wordpress.com/2010/07/07/how-to-make-sem_t-interrupt-safe/)
- `phtread_mutex_t`: deadlocks and recursion
- [why `sem_t` should *not* be used as an atomic counter](https://gustedt.wordpress.com/2010/09/08/sem_t-no-atomic-counter/)
- `sem_t`: premature end of a thread
