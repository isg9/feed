---
title: Why <code>sem_t</code> is not suited as an atomic&nbsp;counter
url: https://gustedt.wordpress.com/2010/09/08/sem_t-no-atomic-counter/
published: "2010-09-08T12:23:24Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=333
---

# Why <code>sem_t</code> is not suited as an atomic&nbsp;counter

C has itself no primitive for atomic access to variables. Whenever we want to deal with some sort of counter in a threaded environment (e.g for reference counting) we have thus to be careful not to erase data that other threads might have written or be just in the course of reading.

### sem\_t as a counter

At a first glance, `sem_t` seems to fit into that need. It has an increment function `sem_post`, a decrement `sem_trywait` and a value test with `sem_getvalue`. If we are careful about interrupt handling (see [this post](https://gustedt.wordpress.com/2010/07/07/how-to-make-sem_t-interrupt-safe/)) this should give us access to a “ *good*” counting device, one might think.

First if we would like to do this an implementation of a reference counter would be relatively tedious. `sem_t` is meant to be waited for if the counter is `0`, but here we would need it the other way round: we would need to wait until the counter falls to `0`. Nevertheless this is doable, not with one semaphore but with several.

### Side effects

What is more annoying is a another defect of `sem_t` its ill-conceived API, namely its tendency to have side effects to the system call. The `sem_trywait` call can be used to decrement a counter without waiting, and in particular to know once the counter has reached `0`. If the counter has been decremented, so it was not `0` before, `sem_trywait` returns `0`, otherwise it returns `-1`.

But the later case has a side effect: `errno` is set to an error code. So in that case, the caller has not only to do what he has to do but also has to do repair work, namely reset `errno` to `0`. This may sound harmless, but it isn’t: `errno` is not a simple variable but an

> expression which expands to a modifiable lvalue that has type int

This must be so, since `errno` is guaranteed to be different per thread, one thread’s `errno` wouldn’t influence the one of another thread. C has no concept of thread-only variables, therefore the system usually as to deploy quite a machinery to achieve a suitable mechanism.

So generally the use of `sem_t` might turn out to be quite inefficient for such a purpose.

### Alternatives

If the API for the function would have gone a different way, namely in returning the precise error code instead of `-1`, this would be much simpler to handle. No access to pseudo-global variables would be necessary, no side effects would take place.

This strategy has been chosen for the other POSIX lock primitives such as `pthread_mutex_t` or `pthread_rwlock_t`.

Both are suitable for the purpose. An implementation with one `pthread_mutex_t`, one `pthread_cond_t` and an `unsigned` should be relatively obvious.

`pthread_rwlock_t` is even directly suitable for a locking structure for resource control. Just have all users of a resource issue a `pthread_rwlock_rdlock` when they start to use it and a `pthread_rwlock_unlock` when they cease to do so. If the resource is unused may then simply tested by a call to `pthread_rwlock_wrlock`.
