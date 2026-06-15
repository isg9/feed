---
title: Exactly-Once Initialization in Asynchronous Python
url: https://nullprogram.com/blog/2020/07/30/
published: "2020-07-30T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/07/30/
---

# Exactly-Once Initialization in Asynchronous Python

## [Exactly-Once Initialization in Asynchronous Python](/blog/2020/07/30/)

July 30, 2020

nullprogram.com/blog/2020/07/30/

*This article was discussed [on Hacker News](https://news.ycombinator.com/item?id=24007354).*

A common situation in [asyncio](https://docs.python.org/3/library/asyncio.html) Python programs is asynchronous initialization. Some resource must be initialized exactly once before it can be used, but the initialization itself is asynchronous — such as an [asyncpg](https://github.com/MagicStack/asyncpg) database. Let’s talk about a couple of solutions.

The naive “solution” would be to track the initialization state in a variable:

```
initialized = False

async def one_time_setup():
    "Do not call more than once!"
    ...

async def maybe_initialize():
    global initialized
    if not initialized:
        await one_time_setup()
        initialized = True

```

The reasoning for `initialized` is the expectation of calling the function more than once. However, if it might be called from concurrent tasks there’s a *race condition*. If the second caller arrives while the first is awaiting `one_time_setup()`, the function will be called a second time.

Switching the order of the call and the assignment won’t help:

```
async def maybe_initialize():
    global initialized
    if not initialized:
        initialized = True
        await one_time_setup()

```

Since asyncio is cooperative, the first caller doesn’t give up control until to other tasks until the `await`, meaning `one_time_setup()` will never be called twice. However, the second caller may return before `one_time_setup()` has completed. What we want is for `one_time_setup()` to be called exactly once, but for no caller to return until it has returned.

### Mutual exclusion

My first thought was to use a [mutex lock](https://docs.python.org/3/library/asyncio-sync.html#lock). This will protect the variable *and* prevent followup callers from progressing too soon. Tasks arriving while `one_time_setup()` is still running will block on the lock.

```
initialized = False
initialized_lock = asyncio.Lock()

async def maybe_initialize():
    global initialized
    async with initialized_lock:
        if not initialized:
            await one_time_setup()
            initialized = True

```

Unfortunately this has a serious downside: **asyncio locks are** **associated with the [loop](https://docs.python.org/3/library/asyncio-eventloop.html) where they were created**. Since the lock variable is global, `maybe_initialize()` can only be called from the same loop that loaded the module. `asyncio.run()` creates a new loop so it’s incompatible.

```
# create a loop: always an error
asyncio.run(maybe_initialize())

# reuse the loop: maybe an error
loop = asyncio.get_event_loop()
loop.run_until_complete((maybe_initialize()))

```

(IMHO, it was a mistake for the asyncio API to include explicit loop objects. It’s a low-level concept that unavoidably leaks through most high-level abstractions.)

A workaround is to create the lock lazily. Thank goodness creating a lock isn’t itself asynchronous!

```
initialized = False
initialized_lock = None

async def maybe_initialize():
    global initialized, initialized_lock
    if not initialized_lock:
        initialized_lock = asyncio.Lock()
    async with initialized_lock:
        if not initialized:
            await one_time_setup()
            initialized = True

```

This is better, but `maybe_initialize()` can still only ever be called from a single loop.

```
asyncio.run(maybe_initialize()) # ok
asyncio.run(maybe_initialize()) # error!

```

### Once

The pthreads API provides [`pthread_once`](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_once.html) to solve this problem. C++11 has similarly has [`std::call_once`](https://en.cppreference.com/w/cpp/thread/call_once). We can build something similar using a future-like object.

```
future = None

async def maybe_initialize():
    global future
    if not future:
        future = asyncio.create_task(one_time_setup())
    await future

```

Awaiting a coroutine more than once is an error, but [tasks](https://docs.python.org/3/library/asyncio-task.html#task-object) are future-like objects and can be awaited more than once. At least on CPython, they can also be awaited in other loops! So not only is this simpler, it also solves the loop problem!

```
asyncio.run(maybe_initialize()) # ok
asyncio.run(maybe_initialize()) # still ok

```

This can be tidied up nicely in a `@once` decorator:

```
def once(func):
    future = None
    async def once_wrapper(*args, **kwargs):
        nonlocal future
        if not future:
            future = asyncio.create_task(func(*args, **kwargs))
        return await future
    return once_wrapper

```

No more need for `maybe_initialize()`, just decorate the original `one_time_setup()`:

```
@once
async def one_time_setup():
    ...

```

- [python](/tags/python/)
- [asyncio](/tags/asyncio/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Exactly-Once%20Initialization%20in%20Asynchronous%20Python) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Exactly-Once+Initialization+in+Asynchronous+Python).

« [Netpbm Animation Showcase](/blog/2020/06/29/)

» [Conventions for Command Line Options](/blog/2020/08/01/)
