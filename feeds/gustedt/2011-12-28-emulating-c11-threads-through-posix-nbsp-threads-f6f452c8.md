---
title: emulating C11 threads through POSIX&nbsp;threads
url: https://gustedt.wordpress.com/2011/12/28/emulating-c11-threads-through-posix-threads/
published: "2011-12-28T00:28:15Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1203
---

# emulating C11 threads through POSIX&nbsp;threads

This month a new version of the C standard, C11, has been published. It brings much less changes to C than C99 and hopefully it will be adopted more easily than that. But it will probably still take some time until the major compilers implement it.

Some of the changes that it brings are minor and can already be simulated within C99 or by extensions that some compilers implement already. Some of them already are emulated by P99. I’ll probably post about them one of these days.

The most important to me seem to be two optional additions to the C library: threads and atomic operations.

Today I want to look into the new threads specification. It comes with basic tools as we are used to it from POSIX threads: functions to control threads, data types and functions to regulate resource access, in particular mutex ( `mtx_t`) and condition ( `cnd_t`). So I was wondering if this thread model could be easily emulated on top of POSIX threads.

I found out that basically it can, but there is one particular interface that is not so direct though it seems at a first glance: thread startup. In POSIX the function pointer that represents the user function to start returns a `void*`. In C11 it returns an `int`. Doesn’t sound much? But it is.

In principle we could cast the function pointer passed in for C11 to the type suitable for POSIX and just call `pthread_create` with it as an argument. But this then would mean that the return from C11 (an `int`, often 32 bit) is silently interpreted as `void*`, often 64 bit nowadays. Calling conventions for functions with different return types need not to be the same, so such an approach could fail badly.

So I have come up with a different solution, that can be visited [here](http://p99.gforge.inria.fr/p99-html/group__threads.html "P99 thread emulation"). The main thing is that the user function `func` (returning the `int`) must be wrapped by another function `wrap` that uses the POSIX model. `wrap` has to receive `func` and the argument `arg` for `func` in its own argument. We also have to find a way to pass the returned `int` through to another thread that eventually joins the created thread. So by that the state variable that we need for each thread already has 4 members:

```
pthread_t id;
int (*func)(void*);
void * arg;
int ret;

```

Then we have to find a way to detach threads. In C11 the only way to have detached threads is to detach them manually. There is no such thing like a thread attribute on startup. Threads have shared state that must be freed after a thread terminates. In the simple thread model of C11 there are not much choices where this deallocation must take place:

- in the thread that joins the terminating thread, if it is not detached
- in the terminating thread itself, if it is detached

So there must be mutual knowledge in the created thread and in the calling thread about the fact if it is detached (callee receives information from the caller) or if it is already finished (caller receives information from the callee). I used an `atomic_flag` for that purpose, one of the new atomic types that comes with C11:

```
atomic_flag detached;

```

Another issue when trying to implement this were memory leaks. By combining the two thread models, valgrind had a hard time to figure out when a thread was really finished and to see if the resources were really released. This was particularly difficult when using `thrd_exit` for which I used `pthread_exit` at first. So I decided to use `setjmp/longjmp` to just jump back into the wrapper function `wrap` whenever we exit the thread through `thrd_exit`:

```
jmp_buf jmp;

```

So after all the state that we have to keep for each thread is relatively complicated. To save some space I used a `union` that overlays those parts of the state that are not used at the same time in the life cycle of a thread.

```
struct p00_thrd {
  pthread_t id;
  int ret;
  atomic_flag detached;
  union {
    struct {
      thrd_start_t func;
      void *arg;
    } init;
    jmp_buf jmp;
  } ovrl;
};

```

Hopefully when the first “native” implementations of the thread part of the library come out they will be able to avoid that blowup. In any case, with P99 you can now start to use the [C11 interface for threads if you are on a POSIX system](http://p99.gforge.inria.fr/p99-html/group__threads.html). For the moment this implementation still uses two specific gcc features:

1. Atomic operations that are provided via gcc builtins. I will discuss this feature in one of my next posts.
2. Folding of variables with only tentative definitions that are used in different compilation units. This is just used to avoid that we have to create a library for the thread local state variable. This can easily be circumvented by declaring that variable `extern` and instantiating it somewhere.
