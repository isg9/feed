---
title: Simplest Program Showing the Difference Between Sequential Consistency and TSO
url: https://blog.regehr.org/archives/898
published: "2013-02-07T19:46:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=898
---

# Simplest Program Showing the Difference Between Sequential Consistency and TSO

The effects of [weak memory models](http://en.wikipedia.org/wiki/Memory_ordering) on programs running on shared-memory multiprocessors can be very difficult to understand. Today I was looking for the simplest program that illustrates the difference between sequentially consistent execution and TSO (the memory model provided by x86 and x86-64). Based on an example from Section 8.2.3.4 of the [big Intel reference manual](http://download.intel.com/products/processor/manual/325462.pdf), I came up with this:

```
#include <pthread.h>
#include <stdio.h>
#include <assert.h>

volatile int x, y, tmp1, tmp2;

void *t0 (void *arg)
{
  x = 1;
  tmp1 = y;
  return 0;
}

void *t1 (void *arg)
{
  y = 1;
  tmp2 = x;
  return 0;
}

int main (void)
{
  while (1) {
    int res;
    pthread_t thread0, thread1;
    x = y = tmp1 = tmp2 = 0;
    res = pthread_create (&thread0, NULL, t0, NULL);
    assert (res==0);
    res = pthread_create (&thread1, NULL, t1, NULL);
    assert (res==0);
    res = pthread_join (thread0, NULL);
    assert (res==0);
    res = pthread_join (thread1, NULL);
    assert (res==0);
    printf ("%d %d\n", tmp1, tmp2);
    if (tmp1==0 && tmp2==0) break;
  }
  return 0;
}

```

It is easy to see that this program cannot terminate on a sequentially consistent multiprocessor (assuming that all system calls succeed). On the other hand, the program can and does terminate under TSO. Because individual processors are sequentially consistent, the program will not terminate on a multicore Linux machine if you pin it to a single core:

> ```
> $ gcc -O tso.c -o tso -pthread
> $ taskset -c 1 ./tso
>
> ```

It’s easy to “fix” this code so that it does not terminate in the multicore case by adding one memory fence per thread. In GCC on x86 and x86-64 a memory fence is:

> ```
> asm volatile ("mfence");
> ```

In some cases this version is preferable since it also stops some kinds of reordering done by the compiler:

> ```
> asm volatile ("mfence" ::: "memory");
> ```

The stronger version should be unnecessary here: since the relevant reads and writes are to volatile-qualified objects, the compiler is already not allowed to reorder them across sequence points.

Thinking abstractly about this stuff always messes with my head; it’s better to just write some code. Also I recommend Section 8.2 of Intel manual, it contains a lot of excellent material.
