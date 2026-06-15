---
title: concurrent ml versus go — wingolog
url: https://wingolog.org/archives/2016/09/20/concurrent-ml-versus-go
published: "2016-09-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/09/20/concurrent-ml-versus-go
---

# concurrent ml versus go — wingolog

## [concurrent ml versus go](/archives/2016/09/20/concurrent-ml-versus-go)

20 September 2016 9:33 PM

- [concurrency](/tags/concurrency)
- [go](/tags/go)
- [ml](/tags/ml)
- [ignorance](/tags/ignorance)
- [guile](/tags/guile)

Peoples! Lately I've been navigating the guile-ship through waters unknown. This post is something of an echolocation to figure out where the hell this ship is and where it should go.

Concretely, I have been working on getting a nice [lightweight concurrency system](https://github.com/wingo/fibers/wiki/Manual) rolling for Guile. I'll write more about that later, but you can think of it as being modelled on Go, though built as a library. (I had previously described it as "Erlang-like", but that's just not accurate.)

Earlier this year at [Curry On](http://curry-on.org/2016/) this topic was burning in my mind and of course when I saw the language-hacker fam there I had to bend their ears. My targets: Matthew Flatt, the amazing boundary-crossing engineer, hacker, teacher, researcher, and implementor of [Racket](http://racket-lang.org/), and Matthias Felleisen, the godfather of the PLT research family. I saw them sitting together and I thought, you know what, what can they have to say to each other? These people have been talking together for 30 years right? Surely they are actually waiting for some ignorant dude to saunter up to the PL genius bar, right?

So saunter I do, saying, "if someone says to you that they want to build a server that will handle 100K or so simultaneous connections on Racket, what abstraction do you tell them to use? Racket threads?" Apparently: yes. A definitive yes, in the case of Matthias, with a pointer to [Robby Findler's paper on kill-safe abstractions](https://www.cs.utah.edu/plt/publications/pldi04-ff.pdf); and still a yes from Matthew with the caveat that for the concrete level of concurrency that I described, you'd have to run tests. More fundamentally, I was advised to look at Concurrent ML (on which Racket's concurrency facilities were based), that CML was much better put together than many modern variants like Go.

This was very interesting and new to me. As y'all probably know, I don't have a formal background in programming languages, and although I've read a lot of literature, reading things only makes you aware of the growing dimension of the not-yet-read. Concurrent ML was even beyond my not-yet-read horizon.

So I went back and read a bunch of papers. Turns out Concurrent ML is like Lisp in that it has a tribe and a tightly-clutched history and a diaspora that reimplements it in whatever language they happen to be working in at the moment. Kinda cool, and, um... a bit hard to appreciate in the current-day context when the only good references are papers from 10 or 20 years ago.

However, after reading a bunch of [John Reppy papers](https://scholar.google.fr/citations?user=hz1S-asAAAAJ&hl=en&oi=sra), here is my understanding of what Concurrent ML is. I welcome corrections; surely I am getting this wrong.

1\. CML is like Go, composed of channels and goroutines. (Forgive the modern referent; I assume most folks know Go at this point.)

2\. Unlike Go, in CML a channel is never buffered. To make a buffered channel in CML, you spawn a thread that manages a buffer between two channels.

3\. Message send and receive operations in CML are built on a lower-level primitive called "events". `(send ch x)` is instead euivalent to `(sync (send-event ch x))`. It's like an event is the derivative of a message send with respect to time, or something.

4\. Events can be combined and transformed using the `choose` and `wrap` combinators.

5\. Doing a `sync` on an event created by `choose` allows a user to build `select` in "user-space", as a library. Cool stuff. So this is what events are for.

6\. There are separate event type implementations for timeouts, channel send/recv blocking operations, file descriptor blocking operations, syscalls, thread joins, and the like. These are supported by the CML implementation.

7\. The early implementations of Concurrent ML were concurrent but not parallel; they did not run multiple "goroutines" on separate CPU cores at the same time. It was only in like 2009 that people started to do CML in parallel. I do not know if this late parallelism has a practical impact on the viability of CML.

**ok go**

What is the relationship of CML to Go? Specifically, is CML more expressive than Go? (I assume the reverse is not the case, but that would also be an interesting result!)

There are a few languages that only allow you to `select` over message receives (not sends), but [Go's `select` doesn't have this limitation](https://golang.org/ref/spec#Select_statements), so that's not a differentiator.

Some people say that it's nice to have events as the common denominator, but I don't get this argument. If the only event under consideration is message send or receive over a channel, events + `choose` \+ `sync` is the same in expressive power as a built-in `select`, as far as I can see. If there are other events, then your runtime already has to support them either way, and something like `(let ((ch (make-channel))) (spawn-fiber (lambda () (put-message ch exp))) (get-message ch))` should be sufficient for any runtime-supported event in *exp*, like sleeps or timeouts or thread joins or whatever.

To me it seems like Go has made the right choices here. I do not see the difference, and that's why I wrote all this, is to be shown the error of my ways. Choosing channels, send, receive, and `select` as the primitives seems to have the same power as SML events.

Let this post be a pentagram on the floor, then, to summon the CML cognoscenti. Well-actuallies are very welcome; hit me up in the comments!

*\[edit: Sam Tobin-Hochstadt tells me I got it wrong and I believe him :) In the meantime while I work out how I was wrong, examples are welcome!\]*

## related articles

- [is go an acceptable cml?](/archives/2016/09/21/is-go-an-acceptable-cml)
- [An incomplete history of language facilities for concurrency](/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [on taking advantage of ragged stops](/archives/2024/09/06/on-taking-advantage-of-ragged-stops)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)

### 6 responses

1. kamstrup says:[21 September 2016 6:20 AM](#ea31bfe32a8fee342bdc014baf857f8c47b969e0)

   One thing where Go's select immediately falls short in your example with N connections is that you can't naturally select{} on a set of channels not known at compile time. Select in Go is a language construct and not a function; which seems to be an issue I run into pretty much every time I use it\[1\]. Maybe it's just me.

   \[1\]: I'm aware that there is helper API for dynamic selects in Go, but it feels clumsy, not to mention odd, that there are two different ways to do what is conventionally not distinct (select() in C works identically in both cases).

2. Vesa Karvonen says:[21 September 2016 7:19 AM](#2a65df52fda7ed2cd2f31a55ff530becfc21d819)

   As said in an earlier comment, CML's choose allows choice over a run-time generated set of operations, which is sometimes necessary. AFAIK, this is not supported by Go. An example would be the selectable queue example from kill safe abstractions paper.

   What CML events give you over Go style select over channel ops is a way to encapsulate protocols as events. An event can include a preparation step (e.g. send message to server), the actual synchronizing operation (e.g. get result message from server), and an after step (e.g. map over result).

   Sometimes servers need to be notified of cancellation and the negative acknowledgment feature of CML allows you to encapsulate that also as part of an event.

   Assuming your select allows to make an exclusive choice over a run-time generated set of get/put operations, allows multiple operations per channel, and allows you to determine which operation was selected, then you can implement CML style events on top with relative ease. Clojure's core.async doesn't quite pass those requirements, but here is an almost full POC of CML style events on top of core.async:

   https://github.com/polytypic/poc.cml

3. srean says:[21 September 2016 3:12 PM](#c8cd47bb282540d10e5a69b7d5d423dc869a087f)

   Concurrency related abstractions in Felix (quite a bit older than Go) are quite relevant to this dicussion http://felix-lang.org/share/src/web/tut/fibres\_02.fdoc http://felix-lang.org/share/src/web/tut/fibres\_04.fdoc

4. Richard says:[21 September 2016 3:45 PM](#4cfcde9b749a9244fc3ae1d67f6c0c4d4a9a659b)

   You might be interested in reading about Occam (and the Transputer hardware that ran it.)

   That was the original concurrent (and hardware parallel) system, with an incredibly simple, innovative, and elegant design. (Arguably we went backwards by decades since then.)

   https://en.wikipedia.org/wiki/Occam\_(programming\_language)

   Xmos (Xmos.com) is an modern derivative of the Transputer from perhaps the key person behind it.

5. Aram Hăvărneanu says:[22 September 2016 11:06 AM](#dfdfe6600ca5f0eee52ba4d881b3a0030878991c)

   \> CML's choose allows choice over a run-time generated set of operations, which is sometimes necessary. AFAIK, this is not supported by Go.

   It is supported by Go: https://golang.org/pkg/reflect/#Select

6. solrize says:[23 September 2016 10:43 PM](#ca3d8e63eec85bc10482e8a3c3cd3940391b7eaa)

   You should look at the paper on the current GHC I/O system if you haven't yet:

   http://haskell.cs.yale.edu/wp-content/uploads/2013/08/hask035-voellmy.pdf

Comments are closed.
