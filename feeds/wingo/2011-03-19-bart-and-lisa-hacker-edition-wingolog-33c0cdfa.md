---
title: bart and lisa, hacker edition — wingolog
url: https://wingolog.org/archives/2011/03/19/bart-and-lisa-hacker-edition
published: "2011-03-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/03/19/bart-and-lisa-hacker-edition
---

# bart and lisa, hacker edition — wingolog

## [bart and lisa, hacker edition](/archives/2011/03/19/bart-and-lisa-hacker-edition)

19 March 2011 9:43 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [ocap](/tags/ocap)
- [rees](/tags/rees)
- [racket](/tags/racket)
- [cloud](/tags/cloud)
- [gnu](/tags/gnu)

Bart and Lisa are both hacking on a computer run by Marge. Bart has a program that sorts a list of integers. Being a generous person, he shares it with Lisa. Lisa would like to use Bart's program, but she doesn't trust Bart -- she wants to make sure that the program is safe to run before running it. She would like to sort a list of credit card numbers and would be quite vexed indeed if Bart's sort procedure posted them all to a web site.

(This example and much of the discussion is taken from the excellent [Rees thesis](http://mumble.net/~jar/pubs/secureos/secureos.html), which I highly, highly recommend.)

One approach she can take is to examine the program, and only run it if it is obviously harmless. Due to fundamental concerns like undecidability, this will be a conservative evaluation. Then if the program is deemed safe, it can be compiled and invoked directly, without having to put it in a sandbox.

The question I wish to discuss this evening is how to do this safe compilation in Guile, in the case in which a Guile program provides environments for Bart and Lisa to both hack on, at the same time, along with means for Bart and Lisa to communicate.

Let us imagine that Bart gives Lisa a program, as a string. The first thing to do is to read it in as a Scheme data structure. Lisa creates a string port, and calls the system's `read` procedure on the port. This produces a Scheme value.

Lisa trusts that the `read` procedure and the `open-input-string` procedures are safe to call. Indeed she has to trust many things from the system; she doesn't really have much of a choice. She has to trust Marge. In this case, barring bugs, the choice is mostly valid. The exception would be reader macros, which cause arbitrary code to run at read-time. Lisa must assume that Bart has not invoked [`read-hash-extend`](http://www.gnu.org/software/guile/manual/html_node/Reader-Extensions.html) on her machine. Indeed, Marge cannot supply `read-hash-extend` to Bart or to Lisa, just as she would not give them access to the low-level [foreign function interface](http://www.gnu.org/software/guile/manual/html_node/Void-Pointers-and-Byte-Access.html).

This brief discussion indicates that there exist in Guile cross-cutting, non-modular facilities which cannot be given to users. If you want to create a secure environment in which programs may only use the [capabilities](http://erights.org/) they are provided, "user-level" environments must not include some routines, like `read-hash-extend`.

**names come to have meanings**

So, having proceeded on, Lisa reads a Scheme form which she can pretty-print to her console, and it looks like this:

```
(define sort
  (lambda (l <)
    (define insert
      (lambda (x sorted)
        (if (null? sorted)
            (list x)
            (if (< x (car sorted))
                (cons x sorted)
                (cons (car sorted) (insert x (cdr sorted)))))))
    (if (null? l)
        '()
        (insert (car l) (sort (cdr l) <)))))

```

How can Lisa know if this program is safe to run?

Actually, this question is something of a cart before the horse; what we should ask is, *what does this program mean*? To that question, we have the lambda calculus to answer for, as part of the Scheme language definition.

In this form we have binding forms, bound variable references, free variable references, and a few conditionals and constants. The compiler obtains this information during its expansion of the form. Here we are assuming that Lisa compiles the form in an environment with the conventional bindings for `define`, `lambda`, and `if`.

Of these, the constants, conditionals, lexical binding forms, and bound variable references are all safe.

The only thing we need be concerned about are the free variable references ( `null?`, `list`, `car`, `cdr`, `cons`, and, interestingly, `sort` itself), and the effect of the toplevel definition (of `sort`).

In Scheme, forms are either definitions, expressions, or sequences of forms. Definitions bind names to values, and have no value themselves. Expressions can evaluate to any number of values. So the question here for Lisa is: is she interested in a definition of a `sort` procedure, or a value? If the latter, she must mark this form as unsafe, as it is a definition. If the former, she needs to decide on what it means for Bart to provide a definition to her.

In Guile, top-level definitions and variable references are given meaning by *modules*. Free variables are scoped relative to the module in which the code appears. If Lisa expands the `sort` definition in a fresh module, then all of the free variables will be resolved relative to that module -- including the `sort` reference within the lambda expression.

Lisa can probably give Bart access to all of the primitives that his program calls: `car`, `cons`, etc.

We start to see how the solution works, then: a program is safe if all of its free variables are safe. `if` is safe. `null?` is safe. And so on. Lisa freely provides these resources to Bart (through his program), because she knows that he can't break them, or use them to break other things. She provides those resources through a fresh module, in which she compiles his program. Once compiled, she can invoke Bart's `sort` directly, via `((module-ref bart-module 'sort) my-credit-card-numbers)`, without the need to run Bart's code in any kind of sandbox.

**world enough, and time**

However, there are two resources which Lisa transfers to Bart's program which are not passed as procedure arguments: time, and space.

What if Bart's program had a bug that made it fail to terminate? What if it simply took too much time? In this case, Marge may provide Lisa with a utility, `safe-apply`, which calls Bart's function, but which cancels it after some amount of time. Such a procedure is easy for Marge to implement with `setitimer`. `setitimer` is one of those cross-cutting concerns however which Marge would not want to provide to either Lisa or Bart directly.

The space question is much more difficult, however. Bart's algorithm might cons a lot of memory, but that's probably OK if it's all garbage. However it's tough for Marge to determine what is Lisa's garbage and what is Bart's garbage, and what non-garbage is Lisa's, and what non-garbage is Bart's, given that they may share objects.

In the Guile case, Marge has even fewer resources at her disposal, as the BDW-GC doesn't give you all that much information. If however she can assume that she can have accurate per-thread byte allocation counters, she can give Lisa the tools needed to abort Bart's program if it allocates too much. Similarly, if Marge restricts her program to only one OS thread, she can guesstimate the active heap size. In that way she can give Lisa tools to restrict Bart's program to a certain amount of active memory.

**related**

Note that Marge's difficulties are not unique to her: until just a few weeks ago, the Linux kernel was still vulnerable to fork bombs, which are an instance of denial-of-service through resource allocation (processes, in that case).

Nor are Lisa's, for that matter, considering the number of Android applications that are given access to user data, and then proceed to give it to large corporations, and Android does have a security model for apps. Likewise the `sort` UNIX utility has access to your SSH private key, and it's only the hacker ethic that has kept free software systems in their current remarkably benign state; though, given the weekly Flash Player and Acrobat Reader vulnerabilities, even that is not good enough (for those of us that actually use those programs).

**future**

I'm going to work on creating a "safe evaluator" that Lisa can use to evaluate expressions from Bart, and later call them. The goal is to investigate the applicability of the ocap security model to the needs of an [autonomy- and privacy-preserving distributed computing substrate](//wingolog.org/archives/2010/04/10/towards-a-gnu-autonomous-cloud). In such an environment, Bart should be able to share an "app" (in the current "app store" sense) with all of his friends. His friends could then run the app, but without needing to trust the app maker, Bart, or anyone else. (One precedent would be [Racket's sandboxed evaluators](http://docs.racket-lang.org/reference/Sandboxed_Evaluation.html); though I don't understand all of their decisions.)

The difficulty of this model seems to me to be more on the side of data storage than of local computation. It's tough to build a distributed queue on top of Tahoe-LAFS, much less GNUnet. Amusingly though, this sort of safe evaluation can be a remedy for that: if an app not only stores data but code, and data storage nodes allow apps to run safe mapreduce / indexing operations there, we may regain more proper data structures, built from the leaves on up. But hey, I get ahead of myself. Until next time, happy hacking.

## related articles

- [storage primitives for the distributed web](/archives/2011/03/24/storage-primitives-for-the-distributed-web)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [an opinionated guide to scheme implementations](/archives/2013/01/07/an-opinionated-guide-to-scheme-implementations)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [fibs, lies, and benchmarks](/archives/2019/06/26/fibs-lies-and-benchmarks)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)

### 5 responses

1. Alexander Larsson says:[20 March 2011 11:52 AM](#399f22138a76da02709530ff75e6e874ac51237a)

   That definition of safe seems a bit limited to me. If bart wants to get at the data passed to the sort call there are several externally visible attributes that can be used to pass out the data. For instance cpu use, memory use or number of threads.

   Any or several of these can be used to signal binary data to another process on the machine. Some are even visible on the network. For instance, cpu use affect network service response times.

2. Sam Tobin-Hochstadt says:[20 March 2011 2:01 PM](#93a1bc87f8000ee2007a99e95b9d3dc3e68b14af)

   For more background and explanation about the technology that Racket uses for sandboxing, see this paper:

   Programming Languages as Operating Systems, or Revenge of the Son of the Lisp Machine

   Flatt et al

   ICFP 1999

   http://www.cs.brown.edu/~sk/Publications/Papers/Published/ffkf-mred/

3. [wingo](http://wingolog.org/) says:[20 March 2011 3:07 PM](#6a3268a9504ca076bd51cca480b3e82e5e68c02c)

   Alex, indeed Bart should not be able to know how many threads Lisa is running, or what the CPU use is. Lisa has to trust Marge when Marge says that Bart does not have access to her data, just as Marge trusts the security kernel (in Rees' terms), the chip makers, the physicists, etc.

   But yes, it is disturbing that there are so many side channels on multiuser machines. We had a decade or two in which all machines were personal, but now with mobile phones and apps, not to mention JavaScript in all your Firefox tabs, things are getting complicated again. Secure multiuser computing is tough.

   Sam, thanks for the link. I'll check it out! As you know I write for a mostly free software audience, but not one that's necessarily well-versed in Scheme literature, so I have perhaps a bit more freedom to let my ignorance roam. Thanks for not only being patient with it, but endeavoring to kindly correct it :-)

4. Sam Tobin-Hochstadt says:[24 March 2011 3:54 PM](#dc529e4df70da4e952461e126dce00010c113219)

   I'm always happy to help. :)

   Personally, I think there are all to few PL-heads who care about the broader free software cause, so those of us that do should support each other.

5. Ewan says:[25 March 2011 6:25 PM](#227171b3f87769753277b899521060e8f3ea0dbe)

   There exists a concept of a jail in Free BSD which sets a virtualized environment for a process to run in. So if Lisa ran Bart's process in a jail which had no access to send mail or ~lisa/.ssh/

   If his program tried to get a process listing, it would only see other processes in his jail.

   http://en.wikipedia.org/wiki/FreeBSD\_jail

   NB - I haven't used FreeBSD or jails; I'm just casually aware of it's existence.

Comments are closed.
