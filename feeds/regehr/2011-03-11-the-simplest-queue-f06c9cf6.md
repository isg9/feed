---
title: The Simplest Queue?
url: https://blog.regehr.org/archives/487
published: "2011-03-11T16:35:23Z"
feed: regehr
guid: http://blog.regehr.org/?p=487
---

# The Simplest Queue?

My student Jianjun is proving things about ARM executables that handle interrupts. It’s very difficult work, so when I asked him to write up a “simple” case study where an interrupt and the main context communicate through a ring buffer, I thought it would be helpful if I handed him the simplest possible queue that is at all realistic. Since queues are already pretty simple, there seemed to be only two tricks worth playing: force queue size to be a power of two, and waste one slot to simplify full-queue detection. Here’s my code:

> ```
> #include "q.h"
>
> #if ((QSIZE)<2)||(!((((QSIZE)|((QSIZE)-1))+1)/2==(QSIZE)))
> #error QSIZE must be >1 and also a power of 2
> #endif
>
> #define MASK ((QSIZE)-1)
>
> static int q[QSIZE];
> static int head, tail;
>
> static int
> inc (int x)
> {
>   return (x + 1) & MASK;
> }
>
> int
> full (void)
> {
>   return inc (head) == tail;
> }
>
> int
> mt (void)
> {
>   return head == tail;
> }
>
> int
> enq (int item)
> {
>   if (full ())
>     return 0;
>   q[head] = item;
>   head = inc (head);
>   return 1;
> }
>
> int
> deq (int *loc)
> {
>   if (mt ())
>     return 0;
>   *loc = q[tail];
>   tail = inc (tail);
>   return 1;
> }
>
> int
> qents (void)
> {
>   int s = head - tail;
>   if (s < 0)
>     s += (QSIZE);
>   return s;
> }
>
> ```

The header file simply defines QSIZE and gives prototypes for the functions. Did I miss any good tricks? The code coming out of compilers I have sitting around isn’t quite as clean as I’d have hoped. There’s some redundancy in the code (calling inc() twice in the enqueue and dequeue functions) but eliminating it should be a simple matter for the compiler.
