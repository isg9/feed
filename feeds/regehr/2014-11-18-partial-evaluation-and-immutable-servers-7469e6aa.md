---
title: Partial Evaluation and Immutable Servers
url: https://blog.regehr.org/archives/1197
published: "2014-11-18T17:22:36Z"
feed: regehr
guid: http://blog.regehr.org/?p=1197
---

# Partial Evaluation and Immutable Servers

Although I haven’t figured out exactly what immutability means for a server (I’m probably just being picky) the general idea of rebuilding a system from spec rather than evolving it with one-off hacks is very appealing. Lately I’ve been thinking about what could be accomplished if the system compiler were able to take advantage of certain kinds of immutability. One kind of technique that would be enabled is [partial evaluation](http://www.utdallas.edu/~gupta/courses/apl/neil.pdf). Let’s look at a simple example starting with an integer power function I found on the web:

```
long powi(long x, long n) {
  assert(n >= 0);
  long  p = x, r = 1;
  while (n > 0) {
    if (n % 2 == 1)
      r *= p;
    p *= p;
    n /= 2;
  }
  return r;
}

```

This function compiles to 20+ instructions. On the other hand, the compiler is able to do considerably better for this special case:

```
long cubei(long x) {
  return powi(x, 3);
}

```

GCC’s output:

**```** **cubei:** **movq %rdi, %rax** **imulq %rdi, %rax** **imulq %rdi, %rax** **ret**

**```**

Here the C compiler has partially evaluated powi() with respect to the constant second argument. The assert() is gone, the loop is gone, etc. This is a very simple example. At the other extreme, people like to say that if you partially evaluate an interpreter with respect to a particular input, you get a compiler. Think, for a minute, about what kind of partial evaluator we would need to have in order to specialize a C interpreter with respect to the powi() code in such a way that we could honestly say that we’ve compiled it. The tool that would support this job is not so easy to create.

Ok, back to immutable servers. What we are looking for is programs in our server image that process immutable or constrained inputs. For example we want to try to show that:

- A daemon, say Redis, is always started using the same configuration file

- For a pair of programs that communicate through a pipe, only a small subset of the full set of commands is ever sent

- Only a subset of the OS kernel’s system calls are invoked

- A program (bash, hopefully) is never invoked at all

Next, we partially evaluate the system with respect to these constant or bounded inputs. If we do this properly, we would expect quite a bit of code handling general cases would fall away, leaving only the specific code needed for our server. This is basically just a big global tree-shaking operation.

Why would we do this? There are two reasons to cut away code and data that we don’t need. First, it reduces unnecessary attack surfaces. Second, it makes the resulting images smaller and faster. We can ship them around more easily and they use less RAM while running.

Partial evaluation is a very old idea, and the idea of applying it to systems software is not new either. Here’s a [good piece of work](http://web.cecs.pdx.edu/~walpole/papers/tocs2001.pdf), and here’s [another one](http://www.vlsi.informatik.tu-darmstadt.de/staff/mjung/publications/seus2005.pdf) that I haven’t read carefully, but that seems reasonable at first glance. Why have these approaches not taken the world by storm? My guess is that it’s just difficult to get good results. In many cases we’re going to be dealing with strings and pointers, and it is very common to run into insurmountable problems when trying to reason about the behavior of programs in the presence of strings and pointers. Consider, for example, a Python script that makes a string using stuff it found in a file, stuff it got over the network, and a few regular expressions. What does the string do when we exec() it?

On the other hand, in the last decade or so SAT/SMT/string solvers have become very powerful, as have symbolic execution techniques. The cloud has created use cases for partial evaluation that did not exist earlier. Security is a worse problem than ever. Compilers are much better. Perhaps it’s time to try again. It is clear that we can’t just point the partial evaluator at our Docker image and expect great things. We’ll need to help it understand what parts of the system are immutable and we’ll also need to incrementally refactor parts of the system to make them cooperate with the partial evaluator. Anyway, research isn’t supposed to be easy.

I’ll finish up by mentioning that there’s a different way to get the same benefits, which is to assemble a system out of a collection of components in such a way that you don’t need a brilliant compiler to eliminate code that you didn’t mean to include. Rather, you avoid including that code in the first place. This is more or less [Mirage’s](http://anil.recoil.org/papers/2013-asplos-mirage.pdf) design point. Both approaches seem worth pursuing.
