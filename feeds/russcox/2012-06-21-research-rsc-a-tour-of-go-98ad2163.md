---
title: 'research!rsc: A Tour of Go'
url: https://research.swtch.com/gotour
published: "2012-06-21T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/gotour
---

# research!rsc: A Tour of Go

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# A Tour of Go Russ Cox June 21, 2012 *research.swtch.com/gotour* Posted on Thursday, June 21, 2012.

Last week, I gave a talk about Go at the Boston Google Developers Group meeting. There were some problems with the recording, so I have rerecorded the talk as a screencast and posted it on YouTube.

Here are the answers to questions asked at the end of the talk.

**Q. How does Go work with debuggers?**

To start, both Go toolchains include debugging information that gdb can read in the final binaries, so basic gdb functionality works on Go programs just as it does on C programs.

We’ve talked for a while about a custom Go debugger, but there isn’t one yet.

Many of the programs we want to debug are live, running programs. The [net/http/pprof](http://golang.org/pkg/net/http/pprof/) package provides debugging information like goroutine stacks, memory profiling, and cpu profiling in response to special HTTP requests.

**Q. If a goroutine is stuck reading from a channel with no other references, does the goroutine get garbage collected?**

No. From the garbage collection point of view, both sides of the channel are represented by the same pointer, so it can’t distinguish the receive and send sides. Even if we could detect this situation, we’ve found that it’s very useful to keep these goroutines around, because the program is probably heading for a deadlock. When a Go program deadlocks, it prints all its goroutine stacks and then exits. If we garbage collected the goroutines as they got stuck, the deadlock handler wouldn’t have anything useful to print except "your entire program has been garbage collected".

**Q. Can a C++ program call into Go?**

We wrote a tool called [cgo](http://golang.org/cmd/cgo/) so that Go programs can call into C, and we’ve implemented support for Go in SWIG, so that Go programs can call into C++. In those programs, the C or C++ can in turn call back into Go. But we don’t have support for a C or C++ program—one that starts execution in the C or C++ world instead of the Go world—to call into Go.

The hardest part of the cross-language calls is converting between the C calling convention and the Go calling convention, specifically with the regard to the implementation of segmented stacks. But that’s been done and works.

Making the assumption that these mixed-language binaries start in Go has simplified a number of parts of the implementation. I don’t anticipate any technical surprises involved in removing these assumptions. It’s just work.

**Q. What are the areas that you specifically are trying to improve the language?**

For the most part, I’m not trying to improve the language itself. Part of the effort in preparing Go 1 was to identify what we wanted to improve and do it. Many of the big changes were based on two or three years of experience writing Go programs, and they were changes we’d been putting off because we knew that they’d be disruptive. But now that Go 1 is out, we want to stop changing things and spend another few years using the language as it exists today. At this point we don’t have enough experience with Go 1 to know what really needs improvement.

My Go work is a small amount of fixing bugs in the libraries or in the compiler and a little bit more work trying to improve the performance of what’s already there.

**Q. What about talking to databases and web services?**

For databases, one of the packages we added in Go 1 is a standard [database/sql](http://golang.org/pkg/database/sql/) package. That package defines a standard API for interacting with SQL databases, and then people can implement drivers that connect the API to specific database implementations like SQLite or MySQL or Postgres.

For web services, you’ve seen the support for JSON and XML encodings. Those are typically good enough for ad hoc REST services. I recently wrote a [package for connecting to the SmugMug photo hosting API](http://go.pkgdoc.org/code.google.com/p/rsc/smugmug), and there’s one generic call that unmarshals the response into a struct of the appropriate type, using [json.Unmarshal](http://golang.org/pkg/encoding/json/#Unmarshal). I expect that XML-based web services like SOAP could be framed this way too, but I’m not aware of anyone who’s done that.

Inside Google, of course, we have plenty of services, but they’re based on protocol buffers, so of course there’s a good [protocol buffer library for Go](http://code.google.com/p/goprotobuf/).

**Q. What about generics? How far off are they?**

People have asked us about generics from day 1. The answer has always been, and still is, that it’s something we’ve put a lot of thought into, but we haven’t yet found an approach that we think is a good fit for Go. We’ve talked to people who have been involved in the design of generics in other languages, and they’ve almost universally cautioned us not to rush into something unless we understand it very well and are comfortable with the implications. We don’t want to do something that we’ll be stuck with forever and regret.

Also, speaking for myself, I don’t miss generics when I write Go programs. What’s there, having built-in support for arrays, slices, and maps, seems to work very well.

Finally, we just made this promise about backwards compatibility with the release of Go 1. If we did add some form of generics, my guess is that some of the existing APIs would need to change, which can’t happen until Go 2, which I think is probably years away.

**Q. What types of projects does Google use Go for?**

Most of the things we use Go for I can’t talk about. One notable exception is that Go is an App Engine language, which we announced at I/O last year. Another is [vtocc](http://code.google.com/p/vitess/), a MySQL load balancer used to manage database lookups in YouTube’s core infrastructure.

**Q. How does the Plan 9 toolchain differ from other compilers?**

It’s a completely incompatible toolchain in every way. The main difference is that object files don’t contain machine code in the sense of having the actual instruction bytes that will be used in the final binary. Instead they contain a custom encoding of the assembly listings, and the linker is in charge of turning those into actual machine instructions. This means that the assembler, C compiler, and Go compiler don’t all duplicate this logic. The main change for Go is the support for segmented stacks.

I should add that we love the fact that we have two completely different compilers, because it keeps us honest about really implementing the spec.

**Q. What are segmented stacks?**

One of the problems in threaded C programs is deciding how big a stack each thread should have. If the stack is too small, then the thread might run out of stack and cause a crash or silent memory corruption, and if the stack is too big, then you’re wasting memory. In Go, each goroutine starts with a small stack, typically 4 kB, and then each function checks if it is about to run out of stack and if so allocates a new stack segment that gets recycled once it’s not needed anymore.

Gccgo supports segmented stacks, but it requires support added recently to the new GNU linker, gold, and that support is only implemented for x86-32 and x86-64.

Segmented stacks are something that lots of people have done before in experimental or research systems, but they have never made it into the C toolchains.

**Q. What is the overhead of segmented stacks?**

It’s a few instructions per function call. It’s been a long time since I tried to measure the precise overhead, but in most programs I expect it to be not more than 1-2%. There are definitely things we could do to try to reduce that, but it hasn’t been a concern.

**Q. Do goroutine stacks adapt in size?**

The initial stack allocated for a goroutine does not adapt. It’s always 4k right now. It has been other values in the past but always a constant. One of the things I’d like to do is to look at what the goroutine will be running and adjust the stack accordingly, but I haven’t.

**Q. Are there any short-term plans for dynamic loading of modules?**

No. I don’t think there are any technical surprises, but assuming that everything is statically linked simplified some of the implementation. Like with calling Go from C++ programs, I believe it’s just work.

Gccgo might be closer to support for this, but I don’t believe that it supports dynamic loading right now either.

**Q. How much does the language spec say about reflection?**

The spec is intentionally vague about reflection, but [package
reflect’s API](http://golang.org/pkg/reflect/) is definitely part of the Go 1 definition. Any conforming implementation would need to implement that API. In fact, gc and gccgo do have different implementations of that package reflect API, but then the packages that use reflect like [fmt](http://golang.org/pkg/fmt/) and [json](http://golang.org/pkg/encoding/json/) can be shared.

**Q. Do you have a release schedule?**

We don’t have any fixed release schedule. We’re not keeping things secret, but we’re also not making commitments to specific timelines.

Go 1 was in progress publicly for months, and if you watched you could see the bug count go down and the release candidates announced, and so on.

Right now we’re trying to slow down. We want people to write things using Go, which means we need to make it a stable foundation to build on. Go 1.0.1, the first bug release, was released four weeks after Go 1, and Go 1.0.2 was seven weeks after Go 1.0.1.

**Q. Where do you see Go in five years? What languages will it replace?**

I hope that it will still be at golang.org, that the Go project will still be thriving and relevant. We built it to write the kinds of programs we’ve been writing in C++, Java, and Python, but we’re not trying to go head-to-head with those languages. Each of those has definite strengths that make them the right choice for certain situations. We think that there are plenty of situations, though, where Go is a better choice.

If Go doesn’t work out, and for some reason in five years we’re programming in something else, I hope the something else would have the features I talked about, specifically the Go way of doing interfaces and the Go way of handling concurrency.

If Go fails but some other language with those two features has taken over the programming landscape, if we can move the computing world to a language with those two features, then I’d be sad about Go but happy to have gotten to that situation.

**Q. What are the limits to scalability with building a system with many goroutines?**

The primary limit is the memory for the goroutines. Each goroutine starts with a 4kB stack and a little more per-goroutine data, so the overhead is between 4kB and 5kB. That means on this laptop I can easily run 100,000 goroutines, in 500 MB of memory, but a million goroutines is probably too much.

For a lot of simple goroutines, the 4 kB stack is probably more than necessary. If we worked on getting that down we might be able to handle even more goroutines. But remember that this is in contrast to C threads, where 64 kB is a tiny stack and 1-4MB is more common.

**Q. How would you build a traditional barrier using channels?**

It’s important to note that channels don’t attempt to be a concurrency Swiss army knife. Sometimes you do need other concepts, and the standard [sync](http://golang.org/pkg/sync/) package has some helpers. I’d probably use a [sync.WaitGroup](http://golang.org/pkg/sync/#WaitGroup).

If I had to use channels, I would do it like in the web crawler example, with a channel that all the goroutines write to, and a coordinator that knows how many responses it expects.

**Q. What is an example of the kind of application you’re working on performance for? How will you beat C++?**

I haven’t been focusing on specific applications. Go is still young enough that if you run some microbenchmarks you can usually find something to optimize. For example, I just sped up floating point computation by about 25% a few weeks ago. I’m also working on more sophisticated analyses for things like escape analysis and bounds check elimination, which address problems that are unique to Go, or at least not problems that C++ faces.

Our goal is definitely not to beat C++ on performance. The goal for Go is to be near C++ in terms of performance but at the same time be a much more productive environment and language, so that you’d rather program in Go.

**Q. What are the security features of Go?**

Go is a type-safe and memory-safe language. There are no dangling pointers, no pointer arithmetic, no use-after-free errors, and so on.

You can break the rules by importing package [unsafe](http://golang.org/pkg/unsafe/), which gives you a special type unsafe.Pointer. You can convert any pointer or integer to an unsafe.Pointer and back. That’s the escape hatch, which you need sometimes, like for extracting the bits of a float64 as a uint64. But putting it in its own package means that unsafe code is explicitly marked as unsafe. If your program breaks in a strange way, you know where to look.

Isolating this power also means that you can restrict it. On App Engine you can’t import package unsafe in the code you upload for your app.

I should point out that the current Go implementation does have [data races](http://research.swtch.com/gorace), but they are not fundamental to the language. It would be possible to eliminate the races at some cost in efficiency, and for now we’ve decided not to do that. There are also tools such as [Thread Sanitizer](http://code.google.com/p/data-race-test/wiki/ThreadSanitizerGo) that help find these kinds of data races in Go programs.

**Q. What language do you think Go is trying to displace?**

I don’t think of Go that way. We were writing C++ code before we did Go, so we definitely wanted not to write C++ code anymore. But we’re not trying to displace all C++ code, or all Python code, or all Java code, except maybe in our own day-to-day work.

One of the surprises for me has been the variety of languages that new Go programmers used to use. When we launched, we were trying to explain Go to C++ programmers, but many of the programmers Go has attracted have come from more dynamic languages like Python or Ruby.

**Q. How does Go make it possible to use multiple cores?**

Go lets you tell the runtime how many operating system threads to use for executing goroutines, and then it muxes the goroutines onto those threads. So if you’ve written a program that has four or more goroutines executing simultaneously, you can tell the runtime to use four OS threads and then you’re running on four cores.

We’ve been pleasantly surprised by how easy people find it to write these kinds of programs. People who have not written parallel or concurrent programs before write concurrent Go programs using channels that can take advantage of multiple cores, and they enjoy the experience. That’s more than you can usually say for C threads. Joe Armstrong, one of the creators of Erlang, makes the point that thinking about concurrency in terms of communication might be more natural for people, since communication is something we’ve done for a long time. I agree.

**Q. How does the muxing of goroutines work?**

It’s not very smart. It’s the simplest thing that isn’t completely stupid: all the scheduling operations are O(1), and so on, but there’s a shared run queue that the various threads pull from. There’s no affinity between goroutines and threads, there’s no attempt to make sophisticated scheduling decisions, and there’s not even preemption.

The goroutine scheduler was the first thing I wrote when I started working on Go, even before I was working full time on it, so it’s just about four years old. It has served us surprisingly well, but we’ll probably want to replace it in the next year or so. We’ve been having some discussions recently about what we’d want to try in a new scheduler.

**Q. Is there any plan to bootstrap Go in Go, to write the Go compiler in Go?**

There’s no immediate plan. Go does ship with a Go program parser written in Go, so the first piece is already done, and there’s an experimental type checker in the works, but those are mainly for writing program analysis tools. I think that Go would be a great language to write a compiler in, but there’s no immediate plan. The current compiler, written in C, works well.

I’ve worked on bootstrapped languages in the past, and I found that bootstrapping is not necessarily a good fit for languages that are changing frequently. It reminded me of climbing a cliff and screwing hooks into the cliff once in a while to catch you if you fall. Once or twice I got into situations where I had identified a bug in the compiler, but then trying to write the code to fix the bug tickled the bug, so it couldn’t be compiled. And then you have to think hard about how to write the fix in a way that avoids the bug, or else go back through your version control history to find a way to replay history without introducing the bug. It’s not fun.

The fact that Go wasn’t written in itself also made it much easier to make significant language changes. Before the initial release we went through a handful of wholesale syntax upheavals, and I’m glad we didn’t have to worry about how we were going to rebootstrap the compiler or ensure some kind of backwards compatibility during those changes.

Finally, I hope you’ve read Ken Thompson’s Turing Award lecture, [Reflections on Trusting Trust](http://cm.bell-labs.com/who/ken/trust.html). When we were planning the initial open source release, we liked to joke that no one in their right mind would accept a bootstrapped compiler binary written by Ken.

**Q. What does Go do to compile efficiently at scale?**

This is something that we talked about a lot in early talks about Go. The main thing is that it cuts off transitive dependencies when compiling a single module. In most languages, if package A imports B, and package B imports C, then the compilation of A reads not just the compiled form of B but also the compiled form of C. In large systems, this gets out of hand quickly. For example, in C++ on my Mac, including reads 25,326 lines from 131 files. (C and C++ headers aren't “compiled form,” but the problem is the same.) Go promises that each import reads a single compiled package file. If you need to know something about other packages to understand that package’s API, then the compiled file includes the extra information you need, but only that.

Of course, if you are building from scratch and package A imports B which imports C, then of course C has to be compiled first, and then B, and then A. The import point is that when you go to compile A, you don’t reload C’s object file. In a real program, the dependencies are usually not a chain like this. We might have A1, A2, A3, and so on all importing B. It’s a significant win if none of them need to reread C.

**Q. How do you identify a good project for Go?**

I think a good project for Go is one that you’re excited about writing in Go. Go really is a general purpose programming language, and except for the compiler work, it’s the only language I’ve written significant programs in for the past four years.

Most of the people I know who are using Go are using it for networked servers, where the concurrency features have something contribute, but it’s great for other contexts too. I’ve used it to write a simple mail reader, file system implementations to read old disks, and a variety of other unnetworked programs.

**Q. What is the current and future IDE support for Go?**

I’m not an IDE user in the modern sense, so really I don’t know. We think that it would be possible to write a really nice IDE specifically for Go, but it’s not something we’ve had time to explore. The Go distribution has a [misc](http://golang.org/misc) directory that contains basic Go support for common editors, and there is a [Goclipse project](http://code.google.com/p/goclipse) to write an Eclipse-based IDE, but I don’t know much about those.

The development environment I use, [acme](https://9p.io/sys/doc/acme/acme.pdf), is great for writing Go code, but not because of any custom Go support.

If you have more questions, please consult [these resources](http://golang.org/help/).
