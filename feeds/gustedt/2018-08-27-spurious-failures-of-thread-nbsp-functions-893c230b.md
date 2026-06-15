---
title: Spurious failures of thread&nbsp;functions
url: https://gustedt.wordpress.com/2018/08/27/spurious-failures-of-thread-functions/
published: "2018-08-27T15:39:33Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3678
---

# Spurious failures of thread&nbsp;functions

C11’s thread interface was not very clear on failure conditions that some functions might encounter. It was not clear that wait functions for conditional variables ( `cnd_t`) and tentative locking of mutexes ( `mtx_t`) may fail spuriously, that is with not apparent reason for the caller. By lack of such a specification, it was not clear how C11 threads could be realized by POSIX threads, *e.g.*

Allowing spurious wakeups is particularly important for the wait functions, because it makes implementing the `cnd_t` type much easier, in particular for the special case that the caller of `cnd_signal` or `cnd_broadcast` does not hold the lock on the corresponding mutex. On the other hand, from an application point of view this does not change much. Even without spurious wakeups, a thread that called \` `cnd_wait` \`, e.g, must in any case check the real condition they are interested in.

These considerations lead to changes for three functions, `cnd_timedwait`, `cnd_wait`, and `mtx_trylock`.

Section 7.26.3.5 p2 of C17 now reads (emphasis represents changed text):

> The `cnd_timedwait` atomically unlocks the mutex pointed to by `mtx` and *blocks* until the condition variable pointed to by `cond` is signaled by a call to `cnd_signal` or to `cnd_broadcast`, or until after the `TIME_UTC`-based calendar time pointed to by `ts` *, or until it is unblocked due to an unspecified reason*. When the calling thread becomes unblocked it locks the variable pointed to by `mtx` before it returns. The `cnd_timedwait` requires that the mutex pointed to by `mtx` be locked by the calling thread.

Similar for 7.26.3.6 p2:

> The `cnd_wait` atomically unlocks the mutex pointed to by `mtx` and *blocks* until the condition variable pointed to by `cond` is signaled by a call to `cnd_signal` or to `cnd_broadcast` *, or until it is unblocked due to an unspecified reason*. When the calling thread becomes unblocked it locks the mutex pointed to by `mtx` before it returns. The `cnd_wait` requires that the mutex pointed to by `mtx` be locked by the calling thread.

Finally, a phrase has been added to the end of 7.26.4.5 p3:

> `mtx_trylock` may spuriously fail to lock an unused resource, in
>
> which case it returns `thrd_busy`.
