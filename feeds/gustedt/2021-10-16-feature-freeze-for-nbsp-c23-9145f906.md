---
title: Feature freeze for&nbsp;C23
url: https://gustedt.wordpress.com/2021/10/16/feature-freeze-for-c23/
published: "2021-10-16T10:25:31Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3773
---

# Feature freeze for&nbsp;C23

The dust is now settling and we finally should have [all proposals](http://www.open-std.org/jtc1/sc22/wg14/www/wg14_document_log.htm) for new C23 features. Below you find links to some of the newer ones that I co-authored. Many previous proposals are still open because WG14 only voted in favor and now they have to be revisted before they can be decided.

- Only reserve names of optional functions if necessary. This mitigates the naming explosion in <math.h> and other headers. [http://open-std.org/jtc1/sc22/wg14/www/docs/n2839.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2839.htm)
- Avoid evaluation of `sizeof` for VLA expressions for which the size is already known [http://open-std.org/jtc1/sc22/wg14/www/docs/n2838.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2838.htm)
- Make `call_once` mandatory [http://open-std.org/jtc1/sc22/wg14/www/docs/n2840.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2840.htm)
- Identifier Syntax using Unicode Standard Annex 31 [http://open-std.org/jtc1/sc22/wg14/www/docs/n2836.pdf](http://open-std.org/jtc1/sc22/wg14/www/docs/n2836.pdf)
- Unsequenced functions v3. This adds function attributes for possible optimizations [http://open-std.org/jtc1/sc22/wg14/www/docs/n2825.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2825.htm)
- Pointers and integer types. In particular requires types `[u]intptr_t` [http://open-std.org/jtc1/sc22/wg14/www/docs/n2822.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2822.htm)
- Require exact-width integer type interfaces [http://open-std.org/jtc1/sc22/wg14/www/docs/n2821.htm](http://open-std.org/jtc1/sc22/wg14/www/docs/n2821.htm)
- Function Pointer Types for Pairing Code and Data. This is indendent of the lambda proposals but should be quite usefull for them [http://open-std.org/jtc1/sc22/wg14/www/docs/n2787.pdf](http://open-std.org/jtc1/sc22/wg14/www/docs/n2787.pdf)
