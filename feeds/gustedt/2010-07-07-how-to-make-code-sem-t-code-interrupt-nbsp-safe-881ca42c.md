---
title: how to make <code>sem_t</code> interrupt&nbsp;safe
url: https://gustedt.wordpress.com/2010/07/07/how-to-make-sem_t-interrupt-safe/
published: "2010-07-07T12:53:04Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=140
---

# how to make <code>sem_t</code> interrupt&nbsp;safe

The POSIX semaphore `wait` calls

```
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);

```

can be interrupted at any time, e.g by IO that is delivered or if a process child terminates. In such a case `errno` is set to `EINTR`:

> **ERRORS**
>
> **EINTR** The call was interrupted by a signal handler; see `signal(7)`.
>
> **NOTES**
>
> A signal handler always interrupts a blocked call to one of these functions, regardless of the use
>
> of the `sigaction`(2) `SA_RESTART` flag.

(Side note, `sem_post` is always atomic in that sense and will never return `EINTR`.)

So if we don’t use `sem_t` in a signal handler context and don’t check for `EINTR` on return of the wait, the whole semaphore approach becomes error prone. In particular there are good chances that with small test cases during development everything runs fine, but that then during production byzantine errors occur once in a while that will be very hard to track.

To test for `EINTR` systematically it is easy to write wrappers that just re-issue the corresponding wait whence the return indicates that the call was interrupted.

```
static inline
int sem_wait_nointr(sem_t *sem) {
  while (sem_wait(sem))
    if (errno == EINTR) errno = 0;
    else return -1;
  return 0;
}

static inline
int sem_trywait_nointr(sem_t *sem) {
  while (sem_trywait(sem))
    if (errno == EINTR) errno = 0;
    else return -1;
  return 0;
}

static inline
int sem_timedwait_nointr(sem_t *sem, const struct timespec *abs_timeout) {
  while (sem_timedwait(sem, abs_timeout))
    if (errno == EINTR) errno = 0;
    else return -1;
  return 0;
}

```

Since they use the `inline` keyword, the variants given here are only suitable for C99, but it should be easy for you to adapt them for other contexts. The `inline` here has the advantage that the call can be inlined efficiently at the caller side, basically resulting in the call of the real POSIX function in question plus some conditional jumps.

BTW, the evaluation of `errno` here has some thread magic. The POSIX standard guarantees that it must behave as if it were a local variable for each thread, so we don’t have to worry about concurrent access to it.
