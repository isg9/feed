---
title: A defer mechanism for&nbsp;C
url: https://gustedt.wordpress.com/2020/12/14/a-defer-mechanism-for-c/
published: "2020-12-14T10:27:57Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3746
---

# A defer mechanism for&nbsp;C

An overview of a paper that presents a reference implementation of a `defer` mechanism (co-authored with Robert C. Seacord) has been accepted for publication at [SAC’21](http://www.sigapp.org/sac/sac2021/). The full version is available as a research report on the HAL archive:

[https://hal.inria.fr/hal-03090771/document](https://hal.inria.fr/hal-03090771/document)

It introduces the implementation-side of a C language mechanism for error handling and deferred cleanup adapted from similar features in the Go programming language. Here is a simple example that uses such a feature:

```
guard {
  void * const p = malloc(25);
  if (!p) break;
  defer free(p);
  void * const q = malloc(25);
  if (!q) break;
  defer free(q);
  if (mtx_lock(&mut)==thrd_error) break;
  defer mtx_unlock(&mut);
  // all resources acquired
  // use p, q, and mut until the end of the block
  ...
}
```

This mechanism improves the proximity, visibility, maintainability, robustness, and security of cleanup and error handling over existing language features. The library implementation of the features described by this paper is publicly available under an Open Source License at [https://gustedt.gitlabpages.inria.fr/defer/](https://gustedt.gitlabpages.inria.fr/defer/).

This feature is under consideration for inclusion in the C Standard and has already been discussed on the last WG14 meeting. The main introduction can be found in a paper written together with Alex Gilding, Tom Scogland, Robert C. Seacord, Martin Uecker, and Freek Wiedijk:

[http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2542.pdf](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2542.pdf)
