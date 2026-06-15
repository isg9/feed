---
title: An incomplete history of language facilities for concurrency — wingolog
url: https://wingolog.org/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency
published: "2016-10-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency
---

# An incomplete history of language facilities for concurrency — wingolog

## [An incomplete history of language facilities for concurrency](/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency)

12 October 2016 1:45 PM

- [pl](/tags/pl)
- [concurrency](/tags/concurrency)
- [erlang](/tags/erlang)
- [go](/tags/go)
- [csp](/tags/csp)
- [guile](/tags/guile)
- [fibers](/tags/fibers)
- [callback hell](/tags/callback%20hell)

I have lately been in the market for better concurrency facilities in [Guile](https://gnu.org/s/guile/). I want to be able to write network servers and peers that can gracefully, elegantly, and efficiently handle many tens of thousands of clients and other connections, but without blowing the complexity budget. It's a hard nut to crack.

Part of the problem is implementation, but a large part is just figuring out what to do. I have often thought that modern musicians must be crushed under the weight of recorded music history, but it turns out in our humble field that's also the case; there are as many concurrency designs as languages, just about. In this regard, what follows is an incomplete, nuanced, somewhat opinionated history of concurrency facilities in programming languages, with an eye towards what I should "buy" for the [Fibers](https://github.com/wingo/fibers) library I have been tinkering on for Guile.

\\* \\* \\*

Modern machines have the raw capability to serve hundreds of thousands of simultaneous long-lived connections, but it’s often hard to manage this at the software level. Fibers tries to solve this problem in a nice way. Before discussing the approach taken in Fibers, it’s worth spending some time on history to see how we got here.

One of the most dominant patterns for concurrency these days is “callbacks”, notably in the [Twisted](https://twistedmatrix.com/) library for Python and the [Node.js](https://nodejs.org/) run-time for JavaScript. The basic observation in the callback approach to concurrency is that the efficient way to handle tens of thousands of connections at once is with low-level operating system facilities like `poll` or `epoll`. You add all of the file descriptors that you are interested in to a “poll set” and then ask the operating system which ones are readable or writable, as appropriate. Once the operating system says “yes, file descriptor 7145 is readable”, you can do something with that socket; but what? With callbacks, the answer is “call a user-supplied closure”: a callback, representing the continuation of the computation on that socket.

Building a network service with a callback-oriented concurrency system means breaking the program into little chunks that can run without blocking. Whereever a program could block, instead of just continuing the program, you register a callback. Unfortunately this requirement permeates the program, from top to bottom: you always pay the mental cost of inverting your program’s control flow by turning it into callbacks, and you always incur run-time cost of closure creation, even when the particular I/O could proceed without blocking. It’s a somewhat galling requirement, given that this contortion is required of the programmer, but could be done by the compiler. We Schemers demand better abstractions than manual, obligatory continuation-passing-style conversion.

Callback-based systems also encourage unstructured concurrency, as in practice callbacks are not the only path for data and control flow in a system: usually there is mutable global state as well. Without strong patterns and conventions, callback-based systems often exhibit bugs caused by concurrent reads and writes to global state.

Some of the problems of callbacks can be mitigated by using [“promises”](https://www.promisejs.org/) or other library-level abstractions; if you’re a Haskell person, you can think of this as lifting all possibly-blocking operations into a monad. If you’re not a Haskeller, that’s cool, neither am I! But if your typey spidey senses are tingling, it’s for good reason: with promises, your whole program has to be transformed to return promises-for-values instead of values anywhere it would block.

An obvious solution to the control-flow problem of callbacks is to use threads. In the most generic sense, a thread is a language feature which denotes an independent computation. Threads are created by other threads, but fork off and run independently instead of returning to their caller. In a system with threads, there is implicitly a scheduler somewhere that multiplexes the threads so that when one suspends, another can run.

In practice, the concept of threads is often conflated with a particular implementation, *kernel threads*. Kernel threads are very low-level abstractions that are provided by the operating system. The nice thing about kernel threads is that they can use any CPU that is the kernel knows about. That’s an important factor in today’s computing landscape, where Moore’s law seems to be giving us more cores instead of more gigahertz.

However, as a building block for a highly concurrent system, kernel threads have a few important problems.

One is that kernel threads simply aren’t designed to be allocated in huge numbers, and instead are more optimized to run in a one-per-CPU-core fashion. Their memory usage is relatively high for what should be a lightweight abstraction: some 10 kilobytes at least and often some megabytes, in the form of the thread’s stack. There are ongoing efforts to reduce this for some systems but we cannot expect wide deployment in the next 5 years, if ever. Even in the best case, a hundred thousand kernel threads will take at least a gigabyte of memory, which seems a bit excessive for book-keeping overhead.

Kernel threads can be a bit irritating to schedule, too: when one thread suspends, it’s for a reason, and it can be that user-space knows a good next thread that should run. However because kernel threads are scheduled in the kernel, it’s rarely possible for the kernel to make informed decisions. There are some “user-mode scheduling” facilities that are in development for some systems, but again only for some systems.

The other significant problem is that building non-crashy systems on top of kernel threads is hard to do, not to mention “correct” systems. It’s an embarrassing situation. For one thing, the low-level synchronization primitives that are typically provided with kernel threads, mutexes and condition variables, are not composable. Also, as with callback-oriented concurrency, one thread can silently corrupt another via unstructured mutation of shared state. It’s worse with kernel threads, though: a kernel thread can be interrupted at any point, not just at I/O. And though callback-oriented systems can theoretically operate on multiple CPUs at once, in practice they don’t. This restriction is [sometimes touted as a benefit](https://glyph.twistedmatrix.com/2014/02/unyielding.html) by proponents of callback-oriented systems, because in such a system, the callback invocations have a single, sequential order. With multiple CPUs, this is not the case, as multiple threads can run at the same time, in parallel.

Kernel threads can work. The Java virtual machine does at least manage to prevent low-level memory corruption and to do so with high performance, but still, even Java-based systems that aim for maximum concurrency avoid using a thread per connection because threads use too much memory.

In this context it’s no wonder that there’s a third strain of concurrency: shared-nothing message-passing systems like Erlang. Erlang isolates each thread (called *processes* in the Erlang world), giving each it its own heap and “mailbox”. Processes can spawn other processes, and the concurrency primitive is message-passing. A process that tries receive a message from an empty mailbox will “block”, from its perspective. In the meantime the system will run other processes. Message sends never block, oddly; instead, sending to a process with many messages pending makes it more likely that Erlang will pre-empt the sending process. It’s a strange tradeoff, but it makes sense when you realize that Erlang was designed for network transparency: the same message send/receive interface can be used to send messages to processes on remote machines as well.

No network is truly transparent, however. At the most basic level, the performance of network sends should be much slower than local sends. Whereas a message sent to a remote process has to be written out byte-by-byte over the network, there is no need to copy immutable data within the same address space. The complexity of a remote message send is O(n) in the size of the message, whereas a local immutable send is O(1). This suggests that hiding the different complexities behind one operator is the wrong thing to do. And indeed, given byte read and write operators over sockets, it’s possible to implement remote message send and receive as a process that serializes and parses messages between a channel and a byte sink or source. In this way we get cheap local channels, and network shims are under the programmer’s control. This is the approach that the Go language takes, and is the one we use in Fibers.

Structuring a concurrent program as separate threads that communicate over channels is an old idea that goes back to Tony Hoare’s work on [“Communicating Sequential Processes”](http://usingcsp.com/) (CSP). CSP is an elegant tower of mathematical abstraction whose layers form a pattern language for building concurrent systems that you can still reason about. Interestingly, it does so without any concept of time at all, instead representing a thread’s behavior as a *trace* of instantaneous events. Threads themselves are like functions that unfold over the possible events to produce the actual event trace seen at run-time.

This view of events as instantaneous happenings extends to communication as well. In CSP, one communication between two threads is modelled as an instantaneous event, partitioning the traces of the two threads into “before” and “after” segments.

Practically speaking, this has ramifications in the Go language, which was heavily inspired by CSP. You might think that a channel is just a an [asynchronous queue](https://developer.gnome.org/glib/stable/glib-Asynchronous-Queues.html) that blocks when writing to a full queue, or when reading from an empty queue. That’s a bit closer to the Erlang conception of how things should work, though as we mentioned, Erlang simply slows down writes to full mailboxes rather than blocking them entirely. However, that’s not what Go and other systems in the CSP family do; sending a message on a channel will block until there is a receiver available, and vice versa. The threads are said to “rendezvous” at the event.

Unbuffered channels have the interesting property that you can [`select`](https://tour.golang.org/concurrency/5) between sending a message on channel a or channel b, and in the end only one message will be sent; nothing happens until there is a receiver ready to take the message. In this way messages are really owned by threads and never by the channels themselves. You can of course add buffering if you like, simply by making a thread that waits on either sends or receives on a channel, and which buffers sends and makes them available to receives. It’s also possible to add explicit support for buffered channels, as Go, `core.async`, and many other systems do, which can reduce the number of context switches as there is no explicit buffer thread.

Whether to buffer or not to buffer is a tricky choice. It’s possible to implement singly-buffered channels in a system like Erlang via an explicit send/acknowlege protocol, though it seems difficult to implement completely unbuffered channels. As we mentioned, it’s possible to add buffering to an unbuffered system by the introduction of explicit buffer threads. In the end though in Fibers we follow CSP’s lead so that we can implement the nice `select` behavior that we mentioned above.

As a final point, `select` is OK but is not a great language abstraction. Say you call a function and it returns some kind of asynchronous result which you then have to `select` on. It could return this result as a channel, and that would be fine: you can add that channel to the other channels in your `select` set and you are good. However, what if what the function does is receive a message on a channel, then do something with the message? In that case the function should return a channel, plus a continuation (as a closure or something). If `select` results in a message being received over that channel, then we call the continuation on the message. Fine. But, what if the function itself wanted to `select` over some channels? It could return multiple channels and continuations, but that becomes unwieldy.

What we need is an abstraction over asynchronous operations, and that is the main idea of a CSP-derived system called [“Concurrent ML”](https://courses.cs.washington.edu/courses/cse590p/06sp/reppy93concurrent.pdf) (CML). Originally implemented as a library on top of Standard ML of New Jersey by John Reppy, CML provides this abstraction, which in Fibers is called an *operation* [1](#FOOT1). Calling `send-operation` on a channel returns an operation, which is just a value. Operations are like closures in a way; a closure wraps up code in its environment, which can be later called many times or not at all. Operations likewise can be *performed* [2](#FOOT2) many times or not at all; performing an operation is like calling a function. The interesting part is that you can compose operations via the `wrap-operation` and `choice-operation` combinators. The former lets you bundle up an operation and a continuation. The latter lets you construct an operation that chooses over a number of operations. Calling `perform-operation` on a choice operation will perform one and only one of the choices. Performing an operation will call its `wrap-operation` continuation on the resulting values.

While it’s possible to implement Concurrent ML in terms of Go’s channels and baked-in `select` statement, it’s more expressive to do it the other way around, as that also lets us implement other operations types besides channel send and receive, for example timeouts and condition variables.

---

[1](#DOCF1) CML uses the term *event*, but I find this to be a confusing name. In this isolated article my terminology probably looks confusing, but in the context of the library I think it can be OK. The jury is out though.

[2](#DOCF2) In CML, *synchronized*.

\\* \\* \\*

Well, that's my limited understanding of the crushing weight of history. Note that part of this article is now in the [Fibers manual](https://github.com/wingo/fibers/wiki/Manual).

Thanks very much to Matthew Flatt, Matthias Felleisen, and Michael Sperber for pushing me towards CML. In the beginning I thought its benefits were small and complication large, but now I see it as being the reverse. Happy hacking :)

## related articles

- [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [is go an acceptable cml?](/archives/2016/09/21/is-go-an-acceptable-cml)
- [concurrent ml versus go](/archives/2016/09/20/concurrent-ml-versus-go)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)

### 4 responses

1. gasche says:[12 October 2016 6:19 PM](#4b0f1239f865df8c986cff3412c02febc7f2b610)

   The multicore-OCaml people have been asking the same sort of questions in the last few years, trying to find a reasonable compromise between generality and cost of implementation. They have a wiki page at https://ocaml.io/w/Multicore.

   One interesting thing about their current design (among many others) is that they propose to let users implement their own scheduler in (OCaml) user-space, instead of implementing a fixed scheduler in the runtime. For this they propose the addition to OCaml of "effects handlers", which could be described shortly as a structured (and typed) style for delimited control.

   Fibers is also a "user-space" implementation of concurrency as it is a Guile library, but you do not seem to emphasize the possibility of implementing domain-specific schedulers -- it is not clear by quickly looking at the code for a non-Scheme expert to see whether the design in fact bakes in a specific scheduler implementation. Is this part of your design considerations?

   (Another recent part of the multicore-OCaml work is the choice to implement a \[Reagents\](https://github.com/ocamllabs/reagents) library for lockfree concurrency. Reagents, introduced in \[this 2012 paper by Aaron Turon\](http://www.mpi-sws.org/%7Eturon/reagents.pdf, allow composable construction of lock-free algorithms and data structure. They are more low-level than CML-style primitives (and less expressive: they remain within the subset of algorithms expressible with only atomic locking), but seem to elegantly pander to some performance-justified needs.

2. [wingo](https://wingolog.org/) says:[13 October 2016 8:00 AM](#c222a4c84ebc1999ed6b95f89d46605737d7b0f4)

   Thanks for stopping by, gasche, and thanks for the links :)

   Regarding scheduling, I don't know what to do. One of the reasons I am working on Fibers as a library is that I know I'm going to make a bunch of mistakes and I don't want to yoke Guile users (and maintainers!) to those mistakes for forever. So your question has a trivial answer in that you can just use some other threading library, or fork this one.

   However that's not very satisfactory :) I will look at what OCaml is doing for inspiration. In the beginning it will probably be a simple work-stealing implementation with no priorities.

   The reagents library looks very interesting, thanks! Incidentally in Fibers is lock-free as well, but using a naive approach -- a mixture of some data structures that can only be manipulated by particular threads, and some that are compare-and-swap over persistent data structures. CAS and GC are so nice together.

   Fibers is currently not pre-emptive. I want to change that at some point, though probably as an option -- some people won't want pre-emption I guess.

3. [Ole Laursen](http://people.iola.dk/olau/) says:[13 October 2016 5:22 PM](#691f21af58e34875855b1d00838bb9a01d85790f)

   Have you read this book?

   https://www.amazon.com/Concurrent-Programming-Java%C2%99-Principles-Pattern/dp/0201310090

   It's a bit of an eye opener.

   I'll be reading up on CML, but I think you might be making a mistake here by confounding the need to handle multiple connections with concurrency. The two things should have nothing to do with each other. You should not need concurrency to talk to multiple endpoints.

   Event-based: I think some of the flak event-based systems are getting is somewhat undeserved - many network applications are event-based in nature. Like a GUI, they're waiting for something to happen. So the problem isn't as such the events, it's more the fact that the underlying system is married to the multiple-endpoints-means-multiple-threads idea so the event handler may block if it needs to do something a bit complicated (like replying) and can't be written sequentially without breaking the event loop.

   In the ideal system, I would write my network service with events where it makes sense, and the handlers for those events would then be able to reply etc. without blocking the whole application. I would not have multiple threads or any concurrency, at all times having only one handler running. Unless I really need the concurrency, e.g. for heavy computations, in which case I'd confine the concurrency to a little isolated part. Or split the load between multiple processes, each having no internal concurrency.

   I think other languages are slowly getting there through the await idea.

4. Patrick Logan says:[14 October 2016 1:55 PM](#fe3e3bf133aa4d3da131d9a46ea246fbb375e9a9)

   Linda / tuple spaces / JavaSpaces provides a coordination mechanism which is simple, by value, and expressively able to implement anything from CSP channels to pub/sub and more in a single process or multiple distributed processes, depending on the implementation.

   e.g. see the QIX OS use of "kernel linda"

   https://drive.google.com/file/d/0B0cKsRm-3yprc0VsYzJXeV9VNDQ/view?usp=drivesdk

Comments are closed.
