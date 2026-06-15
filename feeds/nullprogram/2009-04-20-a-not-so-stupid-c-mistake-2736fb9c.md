---
title: A Not So Stupid C Mistake
url: https://nullprogram.com/blog/2009/04/20/
published: "2009-04-20T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/04/20/
---

# A Not So Stupid C Mistake

## [A Not So Stupid C Mistake](/blog/2009/04/20/)

April 20, 2009

nullprogram.com/blog/2009/04/20/

I was reading through a website of " [computer
stupidities](http://www.rinkworks.com/stupid/cs_programming.shtml)" today when I came across this,

```c
if (a)
  {
    /* do something */
    return x;
  }
else if (!a)
  {
    /* do something else */
    return y;
  }
else
  {
    /* do something entirely different */
    return z;
  }
```

This was quickly dismissed as being an obvious beginner mistake. I don't think this can be dismissed so quickly without thinking it through for a moment. Yes, in the example above we will never reach the last condition where we `return z`, but consider the following,

```c
if (a < b)
  printf ("foo\n");
else if (a > b)
  printf ("bar\n");
else if (a == b)
  printf ("baz\n");
else
  printf ("faz\n");
```

The same quick dismissal might drop the last "faz" print statement as being an impossible condition. Can you think of a situation where the program would print "faz"?

Our final condition will be reached if `a` or `b` is equal to `NAN`, which is defined by the [IEEE floating-point standard](http://en.wikipedia.org/wiki/IEEE_floating-point_standard). It is available in C99 from `math.h`. A `NAN` in any of the comparisons above will evaluate to false.

So don't be so quick to dismiss code like this.

- [c](/tags/c/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20A%20Not%20So%20Stupid%20C%20Mistake) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=A+Not+So+Stupid+C+Mistake).

« [SWF Decompression Perl One-liner](/blog/2009/04/18/)

» [Clay Klein Bottle](/blog/2009/04/28/)
