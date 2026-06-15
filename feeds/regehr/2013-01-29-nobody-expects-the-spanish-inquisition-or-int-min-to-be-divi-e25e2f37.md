---
title: Nobody Expects the Spanish Inquisition, or INT_MIN to be Divided by -1
url: https://blog.regehr.org/archives/887
published: "2013-01-29T00:35:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=887
---

# Nobody Expects the Spanish Inquisition, or INT_MIN to be Divided by -1

`INT_MIN % -1` and `INT_MIN / -1` in C/C++ are little gifts that keep on giving. Recently, Xi Wang has been using this construct to [knock over languages](http://kqueue.org/blog/2012/12/31/idiv-dos/) implemented in C/C++. Then today Tavis Ormandy posted an excellent [local DOS for a Windows 8 machine](https://gist.github.com/4658638).

But the fun doesn’t stop there. For one thing, as I wrote a while ago, it’s [not even clear that `INT_MIN % -1` is undefined behavior](https://blog.regehr.org/archives/175) in C99, C++98, or any earlier version. It is explicitly undefined in C11 and C++11.

Since undefined behavior is undefined, we get fun results from programs like this:

```
#include <limits.h>

int foo (int a, int b) {
  return a / b;
}

int main (void) {
  return foo (INT_MIN, -1);
}

```

Now we’ll compile at a few optimization levels using gcc 4.7.2 on either x86 or x86-64 and see what happens:

> **```** **$ gcc -O0 intmin.c -o intmin** **$ ./intmin** **Floating point exception (core dumped)** **$ gcc -O1 intmin.c -o intmin** **$ ./intmin** **Floating point exception (core dumped)** **$ gcc -O2 intmin.c -o intmin** **$ ./intmin** **$**
>
> **```**

Why does it crash at low optimization levels and not at high ones? It happens that `-O2` is the first optimization level where the call to `foo()` is inlined, exposing its guts to constant propagation and folding, with the result that no `idiv` needs to be executed, and thus no exception gets thrown. A recent build of Clang has exactly the same behavior. If we replace `/` with `%`, the behavior is unchanged (under both compilers).

What should we take away from this? A few things:

- If you can’t prove that division and modulus operations in your C/C++ code aren’t going to do these operations, explicit checks need to be added.

- If you are testing a C/C++ program, try to get it to perform these operations, but keep in mind that all such testing can be unreliable due to the effect shown above.

- Use Clang’s new `-fsanitize=integer` mode. It’s not in 3.2 but it’s in trunk and should be in 3.3 when it comes out in a few months. The integer sanitizer is based on the excellent work that [Will Dietz](http://wdtz.org/) has done in getting [IOC](http://embed.cs.utah.edu/ioc/) into LLVM’s trunk.

Here’s what happens:

> **```** **$ clang -fsanitize=integer intmin.c -o intmin** **$ ./intmin** **intmin.c:5:12: runtime error: division of -2147483648 by -1 cannot be represented in type 'int'** **Floating point exception (core dumped)**
>
> **```**

How is this different from the errors above? First, we get the nice diagnostic. Second, we’re guaranteed to get the error regardless of optimization level.

**UPDATE:** A comment on Xi’s post shows how to crash a Bash shell. For example, on my Ubuntu 12.10 machine:

> **```** **$ ($((-2**63/-1)))** **Floating point exception (core dumped)** **$ bash -version** **GNU bash, version 4.2.37(1)-release (x86_64-pc-linux-gnu)** **Copyright (C) 2011 Free Software Foundation, Inc.** **License GPLv3+: GNU GPL version 3 or later**
>
> **This is free software; you are free to change and redistribute it.** **There is NO WARRANTY, to the extent permitted by law.** **$**
>
> **```**

The behavior on Ubuntu 12.04 is the same. I would love to see more examples like this. If you know of one, please leave a comment.
