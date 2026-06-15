---
title: OpenMP and pwrite()
url: https://nullprogram.com/blog/2017/03/01/
published: "2017-03-01T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2017/03/01/
---

# OpenMP and pwrite()

## [OpenMP and pwrite()](/blog/2017/03/01/)

March 01, 2017

nullprogram.com/blog/2017/03/01/

The most common way I introduce multi-threading to [small C
programs](/blog/2015/07/10/) is with OpenMP (Open Multi-Processing). It’s typically used as compiler pragmas to parallelize computationally expensive loops — iterations are processed by different threads in some arbitrary order.

Here’s an example that computes the [frames of a video](/blog/2011/11/28/) in parallel. Despite being computed out of order, each frame is written in order to a large buffer, then written to standard output all at once at the end.

```
size_t size = sizeof(struct frame) * num_frames;
struct frame *output = malloc(size);
float beta = DEFAULT_BETA;

/* schedule(dynamic, 1): treat the loop like a work queue */
#pragma omp parallel for schedule(dynamic, 1)
for (int i = 0; i < num_frames; i++) {
    float theta = compute_theta(i);
    compute_frame(&output[i], theta, beta);
}

write(STDOUT_FILENO, output, size);
free(output);

```

Adding OpenMP to this program is much simpler than introducing low-level threading semantics with, say, Pthreads. With care, there’s often no need for explicit thread synchronization. It’s also fairly well supported by many vendors, even Microsoft (up to OpenMP 2.0), so a multi-threaded OpenMP program is quite portable without `#ifdef`.

There’s real value this pragma API: **The above example would still** **compile and run correctly even when OpenMP isn’t available.** The pragma is ignored and the program just uses a single core like it normally would. It’s a slick fallback.

When a program really *does* require synchronization there’s `omp_lock_t` (mutex lock) and the expected set of functions to operate on them. This doesn’t have the nice fallback, so I don’t like to use it. Instead, I prefer `#pragma omp critical`. It nicely maintains the OpenMP-unsupported fallback.

```
/* schedule(dynamic, 1): treat the loop like a work queue */
#pragma omp parallel for schedule(dynamic, 1)
for (int i = 0; i < num_frames; i++) {
    struct frame *frame = malloc(sizeof(*frame));
    float theta = compute_theta(i);
    compute_frame(frame, theta, beta);
    #pragma omp critical
    {
        write(STDOUT_FILENO, frame, sizeof(*frame));
    }
    free(frame);
}

```

This would append the output to some output file in an arbitrary order. The critical section [prevents interleaving of
outputs](/blog/2016/08/03/).

There are a couple of problems with this example:

1. Only one thread can write at a time. If the write takes too long, other threads will queue up behind the critical section and wait.

2. The output frames will be out of order, which is probably inconvenient for consumers. If the output is seekable this can be solved with `lseek()`, but that only makes the critical section even more important.

There’s an easy fix for both, and eliminates the need for a critical section: POSIX `pwrite()`.

```
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);

```

It’s like `write()` but has an offset parameter. Unlike `lseek()` followed by a `write()`, multiple threads and processes can, in parallel, safely write to the same file descriptor at different file offsets. The catch is that **the output must be a file, not a pipe**.

```
#pragma omp parallel for schedule(dynamic, 1)
for (int i = 0; i < num_frames; i++) {
    size_t size = sizeof(struct frame);
    struct frame *frame = malloc(size);
    float theta = compute_theta(i);
    compute_frame(frame, theta, beta);
    pwrite(STDOUT_FILENO, frame, size, size * i);
    free(frame);
}

```

There’s no critical section, the writes can interleave, and the output is in order.

If you’re concerned about standard output not being seekable (it often isn’t), keep in mind that it will work just fine when invoked like so:

```
$ ./compute_frames > frames.ppm

```

### Windows Portability

I talked about OpenMP being really portable, then used POSIX functions. Fortunately the Win32 `WriteFile()` function has an “overlapped” parameter that works just like `pwrite()`. Typically rather than call either directly, I’d wrap the write like so:

```
#ifdef _WIN32
#define WIN32_LEAN_AND_MEAN
#include

static int
write_frame(struct frame *f, int i)
{
    HANDLE out = GetStdHandle(STD_OUTPUT_HANDLE);
    DWORD written;
    OVERLAPPED offset = {.Offset = sizeof(*f) * i};
    return WriteFile(out, f, sizeof(*f), &written, &offset);
}

#else /* POSIX */
#include

static int
write_frame(struct frame *f, int i)
{
    size_t count = sizeof(*f);
    size_t offset = sizeof(*f) * i;
    return pwrite(STDOUT_FILENO, buf, count, offset) == count;
}
#endif

```

Except for switching to `write_frame()`, the OpenMP part remains untouched.

### Real World Example

Here’s an example in a real program:

[julia.c](https://gist.github.com/skeeto/d7e17bb2aa40907a3405c3933cb1f936)

Notice because of `pwrite()` there’s no piping directly into `ppmtoy4m`:

```
$ ./julia > output.ppm
$ ppmtoy4m -F 60:1 < output.ppm > output.y4m
$ x264 -o output.mp4 output.y4m

```

[output.mp4](/video/?v=julia-256)

- [c](/tags/c/)
- [posix](/tags/posix/)
- [win32](/tags/win32/)
- [optimization](/tags/optimization/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20OpenMP%20and%20pwrite()) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=OpenMP+and+pwrite%28%29).

« [Asynchronous Requests from Emacs Dynamic Modules](/blog/2017/02/14/)

» [Why I've Retired My PGP Keys and What's Replaced It](/blog/2017/03/12/)
