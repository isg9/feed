---
title: stack overflow — wingolog
url: https://wingolog.org/archives/2014/03/17/stack-overflow
published: "2014-03-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/03/17/stack-overflow
---

# stack overflow — wingolog

## [stack overflow](/archives/2014/03/17/stack-overflow)

17 March 2014 11:40 AM

- [stack overflow](/tags/stack%20overflow)
- [stacks](/tags/stacks)
- [guile](/tags/guile)
- [delimited continuations](/tags/delimited%20continuations)
- [scheme](/tags/scheme)

Good morning, gentle hackers. Today's article is about stack representation, how stack representations affect programs, what it means to run out of stack, and that kind of thing. I've been struggling with the issue for a while now in Guile and finally came to a nice solution. But I'm getting ahead of myself; read on for some background on the issue, and details on what Guile 2.2 will do.

**stack limits**

Every time a program makes a call that is not a tail call, it pushes a new frame onto the stack. Returning a value from a function pops the top frame off the stack. Stack frames take up memory, and as nobody has an infinite amount of memory, deep recursion could cause your program to run out of memory. Running out of stack memory is called stack overflow.

Most languages have a terrible stack overflow story. For example, in C, if you use too much stack, your program will exhibit "undefined behavior". If you are lucky, it will crash your program; if are unlucky, it could [crash your car](http://embeddedgurus.com/state-space/2014/02/are-we-shooting-ourselves-in-the-foot-with-stack-overflow/). It's especially bad in C, as you neither know ahead of time how much stack your functions use, nor the stack limit imposed by the user's system, and the stack limit is often quite small relative to the total memory size.

Things are better, but not much better, in managed languages like Python. Stack overflow is usually assumed to throw an exception (though I couldn't find the specification for this), but actually making that happen is tricky enough that [simple programs can cause Python to abort and dump core](http://bugs.python.org/issue6028). And still, like C, Python and most dynamic languages still have a fixed stack size limit that is usually much smaller than the heap.

Arbitrary stack limits would have an unfortunate effect on Guile programs. For example, the following implementation of the inner loop of `map` is clean and elegant:

```
(define (map f l)
  (if (pair? l)
      (cons (f (car l))
            (map f (cdr l)))
      '()))

```

However, if there were a stack limit, that would limit the size of lists that can be processed with this `map`. Eventually, you would have to rewrite it to use iteration with an accumulator:

```
(define (map f l)
  (let lp ((l l) (out '()))
    (if (pair? l)
        (lp (cdr l) (cons (f (car l)) out))
        (reverse out))))

```

This second version is sadly not as clear, and it also allocates twice as much heap memory (once to build the list in reverse, and then again to reverse the list). You would be tempted to use the destructive [linear-update](http://srfi.schemers.org/srfi-1/srfi-1.html#LinearUpdateProcedures) `reverse!` to save memory and time, but then your code would not be continuation-safe -- if f returned again after the map had finished, it would see an out list that had already been reversed. (If you're interested, you might like [this little Scheme quiz](//wingolog.org/archives/2013/11/02/scheme-quiz-time).) The recursive `map` has none of these problems.

**a solution?**

Guile 2.2 will have no stack limit for Scheme code.

When a thread makes its first Guile call, a small stack is allocated -- just one page of memory. Whenever that memory limit would be reached, Guile arranges to grow the stack by a factor of two.

Ideally, stack growth happens via `mremap`, and ideally at the same address in memory, but it might happen via `mmap` or even `malloc` of another memory block. If the stack moves to a different address, we fix up the frame pointers. Recall that right now Guile runs on a virtual machine, so this is a stack just for Scheme programs; we'll talk about the OS stack later on.

Being able to relocate the stack was not an issue for Guile, as we already needed them to implement [delimited continuations](//wingolog.org/archives/2010/02/26/guile-and-delimited-continuations). However, relocation on stack overflow did cause some tricky bugs in the VM, as relocation could happen at more places. In the end it was OK. Each stack frame in Guile has a fixed size, and includes space to make any nested calls; check my [earlier article on the Guile 2.2 VM](//wingolog.org/archives/2013/11/26/a-register-vm-for-guile) for more. The entry point of a function handles allocation of space for the function's local variables, and that's basically the only point the stack can overflow. The few things that did need to point into the stack were changed to be an offset from the stack base instead of a raw pointer.

Even when you grow a stack by a factor of 2, that doesn't mean you immediately take up twice as much memory. Operating systems usually commit memory to a process on a page-by-page granularity, which is usually around 4 kilobytes. Once accessed, this memory is always a part of your process's memory footprint. However, Guile mitigates this memory usage here; because it has to check for stack overflow anyway, it records a "high-water mark" stack usage since the last garbage collection. When garbage collection happens, Guile arranges to return the unused part of the stack to the operating system (using `MADV_DONTNEED`), but without causing the stack to shrink. In this way, the stack can grow to consume up to all memory available to the Guile process, and when the recursive computation eventually finishes, that stack memory is returned to the system.

You might wonder, why not just [allocate enormous stacks, relying on the kernel to page them in lazily as needed](https://mail.mozilla.org/pipermail/rust-dev/2013-November/006314.html)? The biggest part of the answer is that we need to still be able to target 32-bit platforms, and this isn't a viable strategy there. Even on 64-bit, whatever limit you choose is still a limit. If you choose 4 GB, what if you want to map over a larger list? It's admittedly extreme, given Guile's current GC, but not unthinkable. Basically, your stack should be able to grow as big as your heap could grow. The failure mode for the huge-stack case is also pretty bad; instead of getting a failure to grow your stack, which you can handle with an exception, you get a segfault as the system can't page in enough memory.

The other common strategy is "segmented stacks", but the above link covers the downsides of that in Go and Rust. It would also complicate the multiple-value return convention in Guile, where currently multiple values might temporarily overrun the receiver's stack frame.

**exceptional situations**

Of course, it's still possible to run out of stack memory. Usually this happens because of a program bug that results in unbounded recursion, as in:

```
(define (faulty-map f l)
  (if (pair? l)
      (cons (f (car l)) (faulty-map f l))
      '()))

```

Did you spot the bug? The recursive call to `faulty-map` recursed on l, not `(cdr l)`. Running this program would cause Guile to use up all memory in your system, and eventually Guile would fail to grow the stack. At that point you have a problem: Guile needs to raise an exception to unwind the stack and return memory to the system, but the user might have [throw handlers](https://www.gnu.org/software/guile/manual/html_node/Throw-Handlers.html) in place that want to run before the stack is unwound, and we don't have any stack in which to run them.

Therefore in this case, Guile throws an unwind-only exception that does not run pre-unwind handlers. Because this is such an odd case, Guile prints out a message on the console, in case the user was expecting to be able to get a backtrace from any pre-unwind handler.

**runaway recursion**

Still, this failure mode is not so nice. If you are running an environment in which you are interactively building a program while it is running, such as at a REPL, you might want to impose an artificial stack limit on the part of your program that you are building to detect accidental runaway recursion. For that purpose, there is `call-with-stack-overflow-handler`. You run it like this:

```
(call-with-stack-overflow-handler 10000
  (lambda ()              ; body
    (faulty-map (lambda (x) x) '(1 2 3)))
  (lambda ()              ; handler
    (error "Stack overflow!")))

→ ERROR: Stack overflow

```

The body procedure is called in an environment in which the stack limit has been reduced to some number of words (10000, in the above example). If the limit is reached, the handler procedure will be invoked in the dynamic environment of the error. For the extent of the call to the handler, the stack limit and handler are restored to the values that were in place when `call-with-stack-overflow-handler` was called.

Unlike the unwind-only exception that is thrown if Guile is unable to grow its stack, any exception thrown by a stack overflow handler might invoke pre-unwind handlers. Indeed, the stack overflow handler is itself a pre-unwind handler of sorts. If the code imposing the stack limit wants to protect itself against malicious pre-unwind handlers from the inner thunk, it should abort to a prompt of its own making instead of throwing an exception that might be caught by the inner thunk. (Overflow on unwind via inner `dynamic-wind` is not a problem, as the unwind handlers are run with the inner stack limit.)

Usually, the handler should raise an exception or abort to an outer prompt. However if handler does return, it should return a number of additional words of stack space to grant to the inner environment. A stack overflow handler may only ever "credit" the inner thunk with stack space that was available when the handler was instated. When Guile first starts, there is no stack limit in place, so the outer handler may allow the inner thunk an arbitrary amount of space, but any nested stack overflow handler will not be able to consume more than its limit.

I really, really like [Racket's notes on iteration and recursion](http://docs.racket-lang.org/guide/Lists__Iteration__and_Recursion.html#%28part._.Recursion_versus_.Iteration%29), but treating stack memory just like any other kind of memory isn't always what you want. It doesn't make sense to throw an exception on an out-of-memory error, but it does make sense to do so on stack overflow -- and you might want to do some debugging in the context of the exception to figure out what exactly ran away. It's easy to attribute blame for stack memory use, but it's not so easy for heap memory. And throwing an exception will solve the problem of too much stack usage, but it might not solve runaway memory usage. I prefer the additional complexity of having stack overflow handlers, as it better reflects the essential complexity of resource use.

**os stack usage**

It is also possible for Guile to run out of space on the "C stack" -- the stack that is allocated to your program by the operating system. If you call a primitive procedure which then calls a Scheme procedure in a loop, you will consume C stack space. Guile tries to detect excessive consumption of C stack space, throwing an error when you have hit 80% of the process' available stack (as allocated by the operating system), or 160 kilowords in the absence of a strict limit.

For example, looping through `call-with-vm`, a primitive that calls a thunk, gives us the following:

```
(use-modules (system vm vm))

(let lp () (call-with-vm lp))

→ ERROR: Stack overflow

```

Unfortunately, that's all the information we get. Overrunning the C stack will throw an unwind-only exception, because it's not safe to do very much when you are close to the C stack limit.

If you get an error like this, you can either try rewriting your code to use less stack space, or you can [increase Guile's internal C stack limit](http://www.gnu.org/software/guile/docs/master/guile.html/Debug-Options.html). Unfortunately this is a case in which the existence of a limit affects how you would write your programs. The the best thing is to have your code operate without consuming so much OS stack by avoiding loops through C trampolines.

I don't know what will happen when Guile starts to do native compilation. Obviously we can't relocate the C stack, so lazy stack growth and relocation isn't a viable strategy if we want to share the C and Scheme stacks. Still, we need to be able to relocate stack segments for delimited continuations, so perhaps there will still be two stacks, even with native C compilation. We will see.

Well, that's all the things about stacks. Until next time, happy recursing!

## related articles

- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)
- [guile and delimited continuations](/archives/2010/02/26/guile-and-delimited-continuations)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

### 2 responses

1. [Arne Babenhauserheide](http://draketo.de) says:[17 March 2014 3:20 PM](#44be7f01c0a7ed78ad2a4fb3a4623dea0a47ee10)

   Thank you for the insights!

   call-with-stack-overflow-handler sounds pretty useful. And there are definitely interesting times ahead!

2. Patrick Useldinger says:[2 April 2014 11:36 AM](#386d044e69f2948e844918c883ad221d818debe6)

   I've recently run into stack overflows with Guile while porting some Racket code, and increasing the stack size has not helped. There's no urgency to my porting so I can either wait, or adapt my code to be tail-recursive. Is the release date of Guile 2.2 approximately known?

Comments are closed.
