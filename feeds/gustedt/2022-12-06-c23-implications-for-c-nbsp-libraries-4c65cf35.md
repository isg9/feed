---
title: C23 implications for C&nbsp;libraries
url: https://gustedt.wordpress.com/2022/12/06/c23-implications-for-c-libraries/
published: "2022-12-06T16:41:27Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3850
---

# C23 implications for C&nbsp;libraries

\[Edited after final ballot procedure\]

The upcoming standard C23 has a lot of additions to the C library clause. Most of them are small, some of them are big but optional. I have now finished a [a document](https://gustedt.gitlabpages.inria.fr/c23-library/) that summarizes many of the changes. It has some general discussions about the following subjects

- [Unicode support](https://gustedt.gitlabpages.inria.fr/c23-library/#unicode-support)
- [Thread safety of the C library](https://gustedt.gitlabpages.inria.fr/c23-library/#threads)
- [Const-contract of the C library](https://gustedt.gitlabpages.inria.fr/c23-library/#const-contract-of-the-c-library)
- [Changes to integer types](https://gustedt.gitlabpages.inria.fr/c23-library/#changes-to-integer-types)
- [Attributes](https://gustedt.gitlabpages.inria.fr/c23-library/#attributes)

and then lists changes to individual header files of the C library.

This does not contain an detailed description of the changes to the `math.h` header. First, I am really not an expert on that, and second the changes there are quite invasive and we don’t have a diff-file that would clearly list them.
