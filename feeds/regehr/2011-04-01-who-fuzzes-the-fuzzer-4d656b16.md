---
title: Who Fuzzes the Fuzzer?
url: https://blog.regehr.org/archives/501
published: "2011-04-01T20:08:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=501
---

# Who Fuzzes the Fuzzer?

Although it’s fun to act like our tool [Csmith](http://www.cs.utah.edu/~regehr/papers/pldi11-preprint.pdf) is an infallible compiler smashing device, this isn’t really true. Csmith is made of ~40,000 lines of C++, some of it quite complicated and difficult. Csmith probably contains about as many bugs per LOC as your average compiler. So how do we debug the bug-finding tool? The answer is simple: it debugs itself, and also the compilers that we are testing help debug it. Specifically:

- Xuejun put plenty of assertions into the Csmith code.
- The code emitted by Csmith optionally contains a lot of assertions corresponding to dataflow facts; for example, an alias fact might turn into `assert (!p || p==&x)`. Any time one of these checks fails, either Csmith or a compiler is wrong.
- Every time Csmith finds a compiler bug, we try to manually verify that the (reduced) test case is in fact a valid one before submitting it. Sometimes the reduced test case ends up being invalid, at which point we’ve found a bug either in Csmith or in the reducer.
- Just recently we’ve started using Csmith to test some static analysis tools such as [Frama-C](http://frama-c.com/). As expected, we have been finding bugs in these tools. What we didn’t expect is how effective they would be in spotting tricky flaws in Csmith’s output. The explanation, as far as I can tell, is that compilers don’t look very deeply into the semantics of the compiled code, whereas these tools do.

In the end, Csmith seems to have two advantages in its war on compiler bugs. First, it’s not nearly as big as an aggressive compiler. Second, there are a lot of compilers to test and each of them contains plenty of bugs; in contrast, each Csmith bug has to be fixed just once. Therefore, in practice, we find 10 or more compiler bugs for each Csmith bug.
