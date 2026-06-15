---
title: Ease the transition between C&nbsp;standards
url: https://gustedt.wordpress.com/2016/09/07/ease-the-transition-between-c-standards/
published: "2016-09-07T16:22:03Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2600
---

# Ease the transition between C&nbsp;standards

With this post I will start to discuss a series of modifications of the C standard that I have (or will) propose for C2x. As a starting point there is the observation that the passage from one C standard to the next  was not easy in the past. This, neither for implementers of C compilers nor for users, because it was not easy to capture partial improvements. Basically for gcc, e.g., you had to add `-std=c11` or similar, and then test for the version number to see if a particular feature is implemented. This task then was even more complicated because some of the new features are language, and some are library.

This was tedious and error prone, and I think we should avoid such difficulties in the future.

So for the next version I propose to relax the constraints on C standard header files. I think we should add a feature test macro for each header that reflects the C version which is completely implemented by this header.

Most headers of the C standard are currently optional, either because they provide an optional feature such as threads or atomics, or because an environment may be free standing. Having feature test macros implies that we then can require that all headers are mandatory. If the feature is not supported by the platform, the only contents of such a header would be the feature test macro, set to `0`.

It would be nice if the transition to the next standard could just start by this: everybody provide all standard headers, eventually empty but for the test macro.

For the interested you can find my proposal here:

[Mandatory C library headers](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2069.pdf)

[simplify the transition to a new C standard](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2069.pdf)
