---
title: growing fibers — wingolog
url: https://wingolog.org/archives/2017/06/27/growing-fibers
published: "2017-06-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2017/06/27/growing-fibers
---

# growing fibers — wingolog

## [growing fibers](/archives/2017/06/27/growing-fibers)

27 June 2017 10:17 AM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [delimited continuations](/tags/delimited%20continuations)
- [prompts](/tags/prompts)
- [fibers](/tags/fibers)
- [concurrency](/tags/concurrency)

Good day, Schemers!

Over the last 12 to 18 months, as we were preparing for the [Guile 2.2](https://wingolog.org/archives/2017/03/15/guile-2-2-omg) release, I was growing increasingly dissatisfied at not having a good concurrency story in Guile.

I wanted to be able to spawn a million threads on a core, to support highly-concurrent I/O servers, and [Guile's POSIX threads](https://www.gnu.org/software/guile/manual/html_node/Threads.html) are just not the answer. I needed something different, and this article is about the search for and the implementation of that thing.

**on pthreads**

It's worth being specific why POSIX threads are not a great abstraction. One is that they don't compose: two pieces of code that use mutexes won't necessarily compose together. A correct component A that takes locks might call a correct component B that takes locks, and the other way around, and if both happen concurrently you get the classic deadly-embrace deadlock.

POSIX threads are also terribly low-level. Asking someone to build a system with mutexes and cond vars is like building a house with exploding toothpicks.

I want to program network services in a straightforward way, and POSIX threads don't help me here either. I'd like to spawn a million "threads" (scare-quotes!), one for each client, each one just just looping reading a request, computing and writing the response, and so on. POSIX threads aren't the concrete implementation of this abstraction though, as in most systems you can't have more than a few thousand of them.

Finally as a Guile maintainer I have a duty to tell people the good ways to make their programs, but I can't in good conscience recommend POSIX threads to anyone. If someone is a responsible programmer, then yes we can discuss details of POSIX threads. But for a new Schemer? Never. Recommending POSIX threads is malpractice.

**on scheme**

In Scheme we claim to be minimalists. Whether we actually are that or not is another story, but it's true that we have a culture of trying to grow expressive systems from minimal primitives.

It's sometimes claimed that in Scheme, we don't need threads because we have `call-with-current-continuation`, an ultrapowerful primitive that lets us implement any kind of control structure we want. (The name screams for an abbreviation, so the alias `call/cc` is blessed; minimalism is whatever we say it is, right?) Unfortunately it turned out that while `call/cc` can implement any control abstraction, it can't implement any two. [Abstractions built on `call/cc` don't compose!](http://okmij.org/ftp/continuations/against-callcc.html#traps)

Fortunately, there is a way to build powerful control abstractions that do compose. This article covers the first half of composing a concurrency facility out of a set of more basic primitives.

Just to be concrete, I have to start with a simple implementation of an event loop. We're going to build on it later, but for now, here we go:

```
(define (run sched)
  (match sched
    (($ $sched inbox i/o)
     (define (dequeue-tasks)
       (append (dequeue-all! inbox)
               (poll-for-tasks i/o)))
     (let lp ()
       (for-each (lambda (task) (task))
                 (dequeue-tasks))
       (lp)))))

```

This is a scheduler that is a record with two fields, *inbox* and *i/o*.

The *inbox* holds a queue of pending tasks, as thunks (procedure of no arguments). When something wants to enqueue a task, it posts a thunk to the inbox.

On the other hand, when a task needs to wait in some external input or output being available, it will register an event with *i/o*. Typically *i/o* will be a simple combination of an `epollfd` and a mapping of tasks to enqueue when a file descriptor becomes readable or writable. `poll-for-tasks` does the underlying `epoll_wait` call that pulls new I/O events from the kernel.

There are some details I'm leaving out, like when to have `epoll_wait` return directly, and when to have it wait for some time, and how to wake it up if it's sleeping while a task is posted to the scheduler's inbox, but ultimately this is the core of an event loop.

**a long digression**

Now you might think that I'm getting a little far afield from what my goal was, which was threads or fibers or something. But that's OK, let's go a little farther and talk about "prompts". The term "prompt" comes from the experience you get when you work on the command-line:

```
/home/wingo% ./prog

```

I don't know about you all, but I have the feeling that the `/home/wingo%` has a kind of solid reality, that my screen is not just an array of characters but there is a left-hand-side that belongs to the system, and a right-hand-side that's mine. The two parts are delimited by a prompt. Well prompts in Scheme allow you to provide this abstraction within your program: you can establish a program part that's a "system" facility, for whatever definition of "system" suits your purposes, and a part that's for the "user".

In a way, prompts generalize a pattern of system/user division that has special facilities in other programming languages, such as a try/catch block.

```
try {
  foo();
} catch (e) {
  bar();
}

```

Here again I put the "user" code in italics. Some other examples of control flow patterns that prompts generalize would be early exit of a subcomputation, coroutines, and nondeterminitic choice like SICP's `amb` operator. Coroutines is obviously where I'm headed here in the context of this article, but still there are some details to go over.

To make a prompt in Guile, you can use the `%` operator, which is pronounced "prompt":

```
(use-modules (ice-9 control))

(% expr
   (lambda (k . args) #f))

```

The name for this operator comes from Dorai Sitaram's 1993 paper, [Handling Control](http://www.ccs.neu.edu/scheme/pubs/pldi93-sitaram.pdf); it's actually a pun on the `tcsh` prompt, if you must know. Anyway the basic idea in this example is that we run *expr*, but if it aborts we run the `lambda` handler instead, which just returns `#f`.

Really `%` is just syntactic sugar for `call-with-prompt` though. The previous example desugars to something like this:

```
(let ((tag (make-prompt-tag)))
  (call-with-prompt tag
    ;; Body:
    (lambda () expr)
    ;; Escape handler:
    (lambda (k . args) #f)))

```

(It's not quite the same; [`%` uses a "default prompt tag"](https://www.gnu.org/software/guile/manual/html_node/Shift-and-Reset.html). This is just a detail though.)

You see here that `call-with-prompt` is really the primitive. It will call the *body thunk*, but if an abort occurs within the body to the given *prompt tag*, then the body aborts and the *handler* is run instead.

So if you want to define a primitive that runs a function but allows early exit, we can do that:

```
(define-module (my-module)
  #:export (with-return))

(define-syntax-rule (with-return return body ...)
  (let ((t (make-prompt-tag)))
    (define (return . args)
      (apply abort-to-prompt t args))
    (call-with-prompt t
      (lambda () body ...)
      (lambda (k . rvals)
        (apply values rvals)))))

```

Here we define a module with a little `with-return` macro. We can use it like this:

```
(use-modules (my-module))

(with-return return
  (+ 3 (return 42)))
;; => 42

```

As you can see, calling `return` within the body will abort the computation and cause the `with-return` expression to evaluate to the arguments passed to `return`.

But what's up with the handler? Let's look again at the form of the `call-with-prompt` invocations we've been making.

```
(let ((tag (make-prompt-tag)))
  (call-with-prompt tag
    (lambda () ...)
    (lambda (k . args) ...)))

```

With the `with-return` macro, the handler took a first *k* argument, threw it away, and returned the remaining values. But the first argument to the handler is pretty cool: it is the *continuation* of the computation that was aborted, *delimited* by the prompt: meaning, it's the part of the computation between the `abort-to-prompt` and the `call-with-prompt`, packaged as a function that you can call.

If you call the *k*, the delimited continuation, you reinstate it:

```
(define (f)
  (define tag (make-prompt-tag))
  (call-with-prompt tag
   (lambda ()
     (+ 3
        (abort-to-prompt tag)))
   (lambda (k) k)))

(let ((k (f)))
  (k 1))
;; =& 4

```

Here, the `abort-to-prompt` invocation behaved simply like a "suspend" operation, returning the suspended computation `k`. Calling that continuation resumes it, supplying the value 1 to the saved continuation `(+ 3 [])`, resulting in 4.

Basically, when a delimited continuation suspends, the first argument to the handler is a function that can resume the continuation.

**tasks to fibers**

And with that, we just built coroutines in terms of delimited continuations. We can turn our scheduler inside-out, giving the illusion that each task runs in its own isolated fiber.

```
(define tag (make-prompt-tag))

(define (call/susp thunk)
  (define (handler k on-suspend) (on-suspend k))
  (call-with-prompt tag thunk handler))

(define (suspend on-suspend)
  (abort-to-prompt tag on-suspend))

(define (schedule thunk)
  (match (current-scheduler)
    (($ $sched inbox i/o)
     (enqueue! inbox (lambda () (call/susp thunk))))))

```

So! Here we have a system that can run a thunk in a scheduler. Fine. No big deal. But if the thunk calls `suspend`, then it causes an abort back to a prompt. `suspend` takes a procedure as an argument, the `on-suspend` procedure, which will be called with one argument: the suspended continuation of the thunk. We've layered coroutines on top of the event loop.

Guile's virtual machine is a normal register virtual machine with a stack composed of function frames. It's not necessary to do full CPS conversion to implement delimited control, but if you don't, then your virtual machine needs primitive support for `call-with-prompt`, as Guile's VM does. In Guile then, a suspended continuation is an object composed of the slice of the stack captured between the prompt and the abort, and also the slice of the dynamic stack. (Guile keeps a parallel stack for dynamic bindings. Perhaps we should unify these; dunno.) This object is wrapped in a little procedure that uses VM primitives to push those stack frames back on, and continue.

I say all this just to give you a mental idea of what it costs to suspend a fiber. It will allocate storage proportional to the stack depth between the prompt and the abort. Usually this is a few dozen words, if there are 5 or 10 frames on the stack in the fiber.

We've gone from prompts to coroutines, and from here to fibers there's just a little farther to go. First, note that spawning a new fiber is simply scheduling a thunk:

```
(define (spawn-fiber thunk)
  (schedule thunk))

```

Many threading libraries provide a "yield" primitive, which simply suspends the current thread, allowing others to run. We can do this for fibers directly:

```
(define (yield)
  (suspend schedule))

```

Note that the `on-suspend` procedure here is just `schedule`, which re-schedules the continuation (but presumably at the back of the queue).

Similarly if we are reading on a non-blocking file descriptor and detect that we need more input before we can continue, but none is available, we can suspend and arrange for the `epollfd` to resume us later:

```
(define (wait-for-readable fd)
  (suspend
   (lambda (k)
     (match (current-scheduler)
       (($ $sched inbox i/o)
        (add-read-fd! i/o fd
                      (lambda () (schedule k))))))))

```

In Guile you can arrange to install this function as the "current read waiter", causing it to run whenever a port would block. The details are a little gnarly currently; see the [Non-blocking I/O](https://www.gnu.org/software/guile/manual/html_node/Non_002dBlocking-I_002fO.html) manual page for more.

Anyway the cool thing is that I can run any thunk within a `spawn-fiber`, without modification, and it will run as if in a new thread of some sort.

**solid abstractions?**

I admit that although I am very happy with Emacs, I never really took to using the shell from within Emacs. I always have a terminal open with a bunch of tabs. I think the reason for that is that I never quite understood why I could move the cursor over the bash prompt, or into previous expressions or results; it seemed like I was waking up groggily from some kind of dream where nothing was real. I like the terminal, where the only bit that's "mine" is the current command. All the rest is immutable text in the scrollback.

Similarly when you make a UI, you want to design things so that people perceive the screen as being composed of buttons and so on, not just lines. In essence you trick the user, a willing user who is ready to be tricked, into seeing buttons and text and not just weird pixels.

In the same way, with fibers we want to provide the illusion that fibers actually exist. To solidify this illusion, we're still missing a few elements.

One point relates to error handling. As it is, if an error happens in a fiber and the fiber doesn't handle it, the exception propagates out of the fiber, through the scheduler, and might cause the whole program to error out. So we need to wrap fibers in a catch-all.

```
(define (spawn-fiber thunk)
  (schedule
   (lambda ()
     (catch #t thunk
       (lambda (key . args)
         (print-exception (current-error-port) #f key args))))))

```

Well, OK. Exceptions won't propagate out of fibers, yay. In fact in Guile we add another catch inside the print-exception, in case the print-exception throws an exception... Anyway. Cool.

Another point relates to fiber-local variables. In an operating system, each process has a number of variables that are local to it, notably in UNIX we have the umask, the current effective user, the current directory, the open files and what file descriptors they are associated with, and so on. In Scheme we have similar facilities in the form of *parameters*.

Now the usual way that parameters are used is to bind a new value within the extent of some call:

```
(define (with-output-to-string thunk)
  (let ((p (open-output-string)))
    (parameterize ((current-output-port p))
      (thunk))
    (get-output-string p)))

```

Here the `parameterize` invocation established `p` as the current output port during the call to `thunk`. Parameters already compose quite well with prompts; Guile, like Racket, implements the protocol described by Kiselyov, Shan, and Sabry in their [Delimited Dynamic Binding](http://okmij.org/ftp/papers/DDBinding.pdf) paper (well worth a read!).

The one missing piece is that parameters in Scheme are mutable (by default). Normally if you call `(current-input-port)`, you just get the current value of the current input port parameter. But if you pass an argument, like `(current-input-port p)`, then you actually set the current input port to that new value. This value will be in place until we leave some `parameterize` invocation that parameterizes the current input port.

The problem here is that it could be that there's an interesting parameter which some piece of Scheme code will want to just mutate, so that all further Scheme code will use the new value. This is fine if you have no concurrency: there's just one thing running. But when you have many fibers, you want to avoid mutations in one fiber from affecting others. You want some isolation with regards to parameters. In Guile, we do this with the [with-dynamic-state](https://www.gnu.org/software/guile/manual/html_node/Fluids-and-Dynamic-States.html) facility, which isolates changes to the dynamic state (parameters and so on) within the extent of the `with-dynamic-state` call.

```
(define (spawn-fiber thunk)
  (let ((state (current-dynamic-state)))
    (schedule
     (lambda ()
       (catch #t
         (lambda ()
           (with-dynamic-state state thunk))
         (lambda (key . args)
           (print-exception (current-error-port) #f key args))))))

```

Interestingly, `with-dynamic-state` solves another problem as well. You would like for newly spawned fibers to inherit the parameters from the point at which they were spawned.

```
(parameterize ((current-output-port p))
  (spawn-fiber
   ;; New fiber should inherit current-output-port
   ;; binding as "p"
   (lambda () ...)))

```

Capturing the `(current-dynamic-state)` outside the thunk does this for us.

When I made this change in Guile, making sure that `with-dynamic-state` did not impose a [continuation barrier](https://github.com/wingo/fibers/wiki/Manual#Barriers), I ran into a problem. In Guile we implemented [exceptions in terms of delimited continuations and dynamic binding](https://wingolog.org/archives/2010/02/14/sidelong-glimpses). The current stack of exception handlers was a list, and each element included the exceptions handled by that handler, and what prompt to which to abort before running the exception handler. See where the problem is? If we ship this exception handler stack over to a new fiber, then an exception propagating out of the new fiber would be looking up handlers from another fiber, for prompts that probably aren't even on the stack any more.

The problem here is that if you store a heap-allocated stack of current exception handlers in a dynamic variable, and that dynamic variable is captured somehow (say, by a delimited continuation), then you capture the whole stack of handlers, not (in the case of delimited continuations) the delimited set of handlers that were active within the prompt. To fix this, we had to change Guile's exceptions to instead make `catch` just rebind the exception handler parameter to hold the handler installed by the `catch`. If Guile needs to walk the chain of exception handlers, we introduced a new primitive `fluid-ref*` to do so, building the chain from the current stack of parameterizations instead of some representation of that stack on the heap. It's O( *n*), but life is that way sometimes. This way also, delimited continuations capture the right set of exception handlers.

Finally, Guile also supports [asynchronous interrupts](https://www.gnu.org/software/guile/manual/html_node/Asyncs.html). We can arrange to interrupt a Guile process (or POSIX thread) every so often, as measured in wall-clock or process time. It used to be that interrupt handlers caused a continuation barrier, but this is no longer the case, so now we can add pre-emption to a fibers using interrupts.

**summary and reflections**

In Guile we were able to create a solid-seeming abstraction for fibers by composing other basic building blocks from the Scheme toolkit. Guile users can take an abstraction that's implemented in terms of an event loop (any event loop) and layer fibers on top in a way that feels "real". We were able to do this because we have prompts (delimited continuation) and parameters (dynamic binding), and we were able to compose the two. Actually getting it all to work required fixing a few bugs.

In Fibers, we just use delimited continuations to implement coroutines, and then our fibers are coroutines. If we had coroutines as a primitive, that would work just as well. As it is, each suspension of a fiber will allocate a new continuation. Perhaps this is unimportant, given the average continuation size, but it would be comforting in a way to be able to re-use the allocation from the previous suspension (if any). [Other languages with coroutine primitives](https://wingolog.org/archives/2013/02/25/on-generators) might have an advantage here, though delimited dynamic binding is still relatively uncommon.

Another point is that because we use prompts to suspend fiberss, we effectively are always unwinding and rewinding the dynamic state. In practice this should be transparent to the user and the implementor should make this transparent from a performance perspective, with the exception of `dynamic-wind`. Basically any fiber suspension will be run the "out" guard of any enclosing `dynamic-wind`, and resumption will run the "in" guard. In practice we find that we defer "finalization" issues to `with-throw-handler` / `catch`, which unlike `dynamic-wind` don't run on every entry or exit of a dynamic extent and rather just run on exceptional exits. We will see over time if this situation is acceptable. It's certainly another nail in the coffin of `dynamic-wind` though.

This article started with pthreads malaise, and although we've solved the problem of having a million fibers, we haven't solved the communications problem. How should fibers communicate with each other? This is the topic for my next article. Until then, happy hacking :)

## related articles

- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 3 responses

1. [Alex @ Asset plus](http://asset.plus) says:[27 June 2017 4:23 PM](#d2f29d9c1cc1c3c8a10b3a93d52b6e9a16150124)

   Were you able to create 1 million threads concurrently or in a queue-like fashion? Can you share resources between threads without getting into a deadlock situation?

2. [wingo](https://wingolog.org/) says:[29 June 2017 4:31 AM](#4e53fed1c880037ca2cb5500d61737a58338933f)

   I can create 1M threads concurrently, yes.

   As far as communication between threads goes, that's a topic for the next article :)

3. Bengan says:[1 July 2017 8:50 AM](#e9f0003a069c006eba76fa9030e28810ef69a492)

   Another great post! Most of it is way above my head, but I am really hyped about fibers, and I am looking forward to using it in the future!

Comments are closed.
