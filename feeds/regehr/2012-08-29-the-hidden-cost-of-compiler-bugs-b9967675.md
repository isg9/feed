---
title: The Hidden Cost of Compiler Bugs
url: https://blog.regehr.org/archives/782
published: "2012-08-29T17:11:29Z"
feed: regehr
guid: http://blog.regehr.org/?p=782
---

# The Hidden Cost of Compiler Bugs

I have a hypothesis that compiler bugs impose a noticeable but hard-to-measure tax on software development. I’m not talking so much about compiler crashes, although they are annoying, but more about cases where an optimization or code generation bug causes a program to incorrectly segfault or generate a wrong result. Generally, when looking at a test case that triggers one of these bugs, I can’t find any reason why analogous code could not be embedded in some large application. As a random example, earlier this year a development version of GCC would miscompile this code when `a` and `b` have initial value 0:

```
b = (~a | 0 >= 0) & 0x98685255F;
printf ("%d\n", b < 0);

```

The code looks stupid, but such things can easily arise out of macro expansion, constant propagation, function inlining, or a hundred other common transformations. Compiler bugs that are exposed by inlining are pernicious because they may not be triggered during separate compilation, and therefore may not be detectable during unit testing.

The question is: What happens when your code triggers a bug like this? Keep in mind that the symptom is simply that the code misbehaves when the optimization options are changed. In a C/C++ program, by far the most likely cause for this kind of effect is not a compiler bug, but rather one or more undefined behaviors being executed by the program being developed (in the presence of undefined behavior, the compiler is not obligated to behave consistently).

Since tools for detecting undefined behaviors are sorely lacking, at this point debugging the problem usually degenerates to a depressing process of adding break/watchpoints, adding print statements, and messing with the code. My suspicion (and my own experience, both personal and gained by looking over the shoulders of many years’ worth of students) is that much of the time, the buggy behavior goes away as the result of an incidental change, without being fully understood. Thus, the compiler bug goes undetected and lives to bite us another day.

I’m not hopeful that we can estimate the true cost of compiler bugs, but we can reduce the cost through better compiler v&v and by better tools for detecting undefined behaviors.
