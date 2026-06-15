---
title: Asynchronously Opening and Closing Files in Asyncio
url: https://nullprogram.com/blog/2020/09/04/
published: "2020-09-04T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/09/04/
---

# Asynchronously Opening and Closing Files in Asyncio

## [Asynchronously Opening and Closing Files in Asyncio](/blog/2020/09/04/)

September 04, 2020

nullprogram.com/blog/2020/09/04/

Python [asyncio](https://docs.python.org/3/library/asyncio.html) has support for asynchronous networking, subprocesses, and interprocess communication. However, it has nothing for asynchronous file operations — opening, reading, writing, or closing. This is likely in part because operating systems themselves also lack these facilities. If a file operation takes a long time, perhaps because the file is on a network mount, then the entire Python process will hang. It’s possible to work around this, so let’s build a utility that can asynchronously open and close files.

The usual way to work around the lack of operating system support for a particular asynchronous operation is to [dedicate threads to waiting on
those operations](http://docs.libuv.org/en/v1.x/design.html#file-i-o). By using a thread pool, we can even avoid the overhead of spawning threads when we need them. Plus asyncio is designed to play nicely with thread pools anyway.

### Test setup

Before we get started, we’ll need some way to test that it’s working. We need a slow file system. One thought is to [use ptrace to intercept the
relevant system calls](/blog/2018/06/23/), though this isn’t quite so simple. The other threads need to continue running while the thread waiting on `open(2)` is paused, but ptrace pauses the whole process. Fortunately there’s a simpler solution anyway: `LD_PRELOAD`.

Setting the `LD_PRELOAD` environment variable to the name of a shared object will cause the loader to load this shared object ahead of everything else, allowing that shared object to override other libraries. I’m on x86-64 Linux (Debian), and so I’m looking to override `open64(2)` in glibc. Here’s my `open64.c`:

```
#define _GNU_SOURCE
#include
#include
#include

int
open64(const char *path, int flags, int mode)
{
    if (!strncmp(path, "/tmp/", 5)) {
        sleep(3);
    }
    int (*f)(const char *, int, int) = dlsym(RTLD_NEXT, "open64");
    return f(path, flags, mode);
}

```

Now Python must go through my C function when it opens files. If the file resides where under `/tmp/`, opening the file will be delayed by 3 seconds. Since I still want to actually open a file, I use `dlsym()` to access the *real* `open64()` in glibc. I build it like so:

```
$ cc -shared -fPIC -o open64.so open64.c -ldl

```

And to test that it works with Python, let’s time how long it takes to open `/tmp/x`:

```
$ touch /tmp/x
$ time LD_PRELOAD=./open64.so python3 -c 'open("/tmp/x")'

real    0m3.021s
user    0m0.014s
sys     0m0.005s

```

Perfect! (Note: It’s a little strange putting `time` *before* setting the environment variable, but that’s because I’m using Bash and it `time` is special since this is the shell’s version of the command.)

### Thread pools

Python’s standard `open()` is most commonly used as a *context manager* so that the file is automatically closed no matter what happens.

```
with open('output.txt', 'w') as out:
    print('hello world', file=out)

```

I’d like my asynchronous open to follow this pattern using [`async\ with`](https://www.python.org/dev/peps/pep-0492/). It’s like `with`, but the context manager is acquired and released asynchronously. I’ll call my version `aopen()`:

```
async with aopen('output.txt', 'w') as out:
    ...

```

So `aopen()` will need to return an *asynchronous context manager*, an object with methods `__aenter__` and `__aexit__` that both return [*awaitables*](https://docs.python.org/3/glossary.html#term-awaitable). Usually this is by virtue of these methods being [*coroutine functions*](https://docs.python.org/3/glossary.html#term-coroutine-function), but a normal function that directly returns an awaitable also works, which is what I’ll be doing for `__aenter__`.

```
class _AsyncOpen():
    def __init__(self, args, kwargs):
        ...

    def __aenter__(self):
        ...

    async def __aexit__(self, exc_type, exc, tb):
        ...

```

Ultimately we have to call `open()`. The arguments for `open()` will be given to the constructor to be used later. This will make more sense when you see the definition for `aopen()`.

```
    def __init__(self, args, kwargs):
        self._args = args
        self._kwargs = kwargs

```

When it’s time to actually open the file, Python will call `__aenter__`. We can’t call `open()` directly since that will block, so we’ll use a thread pool to wait on it. Rather than create a thread pool, we’ll use the one that comes with the current event loop. The `run_in_executor()` method runs a function in a thread pool — where `None` means use the default pool — returning an asyncio future representing the future result, in this case the opened file object.

```
    def __aenter__(self):
        def thread_open():
            return open(*self._args, **self._kwargs)
        loop = asyncio.get_event_loop()
        self._future = loop.run_in_executor(None, thread_open)
        return self._future

```

Since this `__aenter__` is not a coroutine function, it returns the future directly as its awaitable result. The caller will await it.

The default thread pool is limited to one thread per core, which I suppose is the most obvious choice, though not ideal here. That’s fine for CPU-bound operations but not for I/O-bound operations. In a real program we may want to use a larger thread pool.

Closing a file may block, so we’ll do that in a thread pool as well. First pull the file object [from the future](/blog/2020/07/30/), then close it in the thread pool, waiting until the file has actually closed:

```
    async def __aexit__(self, exc_type, exc, tb):
        file = await self._future
        def thread_close():
            file.close()
        loop = asyncio.get_event_loop()
        await loop.run_in_executor(None, thread_close)

```

The open and close are paired in this context manager, but it may be concurrent with an arbitrary number of other `_AsyncOpen` context managers. There will be some upper limit to the number of open files, so **we need to be careful not to use too many of these things** **concurrently**, something [which easily happens when using unbounded
queues](/blog/2020/05/24/). Lacking back pressure, all it takes is for tasks to be opening files slightly faster than they close them.

With all the hard work done, the definition for `aopen()` is trivial:

```
def aopen(*args, **kwargs):
    return _AsyncOpen(args, kwargs)

```

That’s it! Let’s try it out with the `LD_PRELOAD` test.

### A test drive

First define a “heartbeat” task that will tell us the asyncio loop is still chugging away while we wait on opening the file.

```
async def heartbeat():
    while True:
        await asyncio.sleep(0.5)
        print('HEARTBEAT')

```

Here’s a test function for `aopen()` that asynchronously opens a file under `/tmp/` named by an integer, (synchronously) writes that integer to the file, then asynchronously closes it.

```
async def write(i):
    async with aopen(f'/tmp/{i}', 'w') as out:
        print(i, file=out)

```

The `main()` function creates the heartbeat task and opens 4 files concurrently though the intercepted file opening routine:

```
async def main():
    beat = asyncio.create_task(heartbeat())
    tasks = [asyncio.create_task(write(i)) for i in range(4)]
    await asyncio.gather(*tasks)
    beat.cancel()

asyncio.run(main())

```

The result:

```
$ LD_PRELOAD=./open64.so python3 aopen.py
HEARTBEAT
HEARTBEAT
HEARTBEAT
HEARTBEAT
HEARTBEAT
HEARTBEAT
$ cat /tmp/{1,2,3,4}
1
2
3
4

```

As expected, 6 heartbeats corresponding to 3 seconds that all 4 tasks spent concurrently waiting on the intercepted `open()`. Here’s the full source if you want to try it our for yourself:

[https://gist.github.com/skeeto/89af673a0a0d24de32ad19ee505c8dbd](https://gist.github.com/skeeto/89af673a0a0d24de32ad19ee505c8dbd)

### Caveat: no asynchronous reads and writes

*Only* opening and closing the file is asynchronous. Read and writes are unchanged, still fully synchronous and blocking, so this is only a half solution. A full solution is not nearly as simple because asyncio is async/await. Asynchronous reads and writes would require all new APIs [with different coloring](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/). You’d need an `aprint()` to complement `print()`, and so on, each returning an `awaitable` to be awaited.

This is one of the unfortunate downsides of async/await. I strongly prefer conventional, preemptive concurrency, *but* we don’t always have that luxury.

- [c](/tags/c/)
- [linux](/tags/linux/)
- [python](/tags/python/)
- [asyncio](/tags/asyncio/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Asynchronously%20Opening%20and%20Closing%20Files%20in%20Asyncio) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Asynchronously+Opening+and+Closing+Files+in+Asyncio).

« [Conventions for Command Line Options](/blog/2020/08/01/)

» [w64devkit: (Almost) Everything You Need](/blog/2020/09/25/)
