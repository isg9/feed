---
title: the floating point naming&nbsp;explosion
url: https://gustedt.wordpress.com/2019/09/27/the-floating-point-naming-explosion/
published: "2019-09-27T13:28:50Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3716
---

# the floating point naming&nbsp;explosion

The recent integration of the floating point technical specifications (TS) has had a disastrous effect on the set of reserved identifiers in C. If C2x would go through as it currently stands about 1700 identifiers that were completely unprotected and unannounced would enter the global name space and nasty conflicts with user code would become unavoidable. There have been several attempts in history to introduce scoped identifiers to C, but we can’t wait for such miracles to happen before C2x is supposed to come out.

Therefore, in a paper for WG14 that will be debated in the next meeting in Ithaca, I propose to take emergency measures and rename most of these new identifiers before they hit the public. On a complementary aspect, we propose decent names for the newly introduced floating point types.

N2426 Jens Gustedt, [Contain the floating point naming explosion](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2426.pdf)
