---
title: What's //go:nosplit for?
url: /2025/07/07/nosplit
published: "2025-07-07T00:00:00Z"
feed: mcyoung
guid: /2025/07/07/nosplit
---

# What's //go:nosplit for?

Most people don’t know that Go has special syntax for directives. Unfortunately, it’s not real syntax, it’s just a comment. For example, `//go:noinline` causes the next function declaration to never get inlined, which is useful for changing the inlining cost of functions that call it.

There are three types of directives:

1. The ones documented in [`gc`’s doc comment](https://pkg.go.dev/cmd/compile#hdr-Function_Directives). This includes `//go:noinline` and `//line`.

2. The ones documented elsewhere, such as `//go:build` and `//go:generate`.

3. The ones documented in [runtime/HACKING.md](https://cs.opensource.google/go/go/+/refs/tags/go1.24.4:src/runtime/HACKING.md), which can only be used if the `-+` flag is passed to `gc`. This includes `//go:nowritebarrier`.

4. The ones not documented at all, whose existence can be discovered by searching the compiler’s tests. These include `//go:nocheckptr`, `//go:nointerface`, and `//go:debug`.

We are most interested in a directive of the first type, `//go:nosplit`. According to the documentation:

> [reference](#ref:1)
>
> The `//go:nosplit` directive must be followed by a function declaration. It specifies that the function must omit its usual stack overflow check. This is most commonly used by low-level runtime code invoked at times when it is unsafe for the calling goroutine to be preempted.

What does this even mean? Normal program code can use this annotation, but its behavior is poorly specified. Let’s dig in.

## [Go Stack Growth](\#go-stack-growth)

Go allocates very small stacks for new goroutines, which grow their stack dynamically. This allows a program to spawn a large number of short-lived goroutines without spending a lot of memory on their stacks.

This means that it’s very easy to overflow the stack. Every function knows how large its stack is, and `runtime.g`, the goroutine struct, contains the end position of the stack; if the stack pointer is less than it (the stack grows up) control passes to `runtime.morestack`, which effectively preempts the goroutine while its stack is resized.

In effect, every Go function has the following code around it:

```x86-go highlight
  TEXT    .f(SB), ABIInternal, $24-16
  CMPQ    SP, 16(R14)
  JLS     grow
  PUSHQ   BP
  MOVQ    SP, BP
  SUBQ    $16, SP
  // Function body...
  ADDQ    $16, SP
  POPQ    BP
  RET
grow:
  MOVQ    AX, 8(SP)
  MOVQ    BX, 16(SP)
  CALL    runtime.morestack_noctxt(SB)
  MOVQ    8(SP), AX
  MOVQ    16(SP), BX
  JMP     .f(SB)

```

[x86 Assembly (Go Syntax)](#code:1)

Note that `r14` holds a pointer to the current `runtime.g`, and the stack limit is the third word-sized field ( `runtime.g.stackguard0`) in that struct, hence the offset of 16. If the stack is about to be exhausted, it jumps to a special block at the end of the function that spills all of the argument registers, traps into the runtime, and, once that’s done, unspills the arguments and re-starts the function.

Note that arguments are spilled *before* adjusting `rsp`, which means that the arguments are written to the *caller’s* stack frame. This is part of Go’s ABI; callers must allocate space at the top of their stack frames for any function that they call to spill all of its registers for preemption[1](#fn:1).

Preemption is not reentrant, which means that functions that are running in the context of a preempted G or with no G at all must not be preempted by this check.

## [Nosplit Functions](\#nosplit-functions)

The `//go:nosplit` directive marks a function as “nosplit”, or a “non-splitting function”. “Splitting” has nothing to do with what this directive does.

> [aside](#aside:1)
>
> In the bad old days, Go’s stacks were *split* up into *segments*, where each segment ended with a pointer to the next, effectively replacing the stack’s single array with a linked list of such arrays.
>
> Segmented stacks were terrible. Instead of triggering a resize, these prologues were responsible for updating `rsp` to the next (or previous) block by following this pointer, whenever the current segment bottomed out. This meant that if a function call happened to be on a segment boundary, it would be *extremely slow* in comparison to other function calls, due to the significant work required to update `rsp` correctly.
>
> This meant that unlucky sizing of stack frames meant sudden performance cliffs. Fun!
>
> Go has since figured out that segmented stacks are a terrible idea. In the process of implementing a correct GC stack scanning algorithm (which it did not have for many stable releases), it also gained the ability to copy the contents of a stack from one location to another, updating pointers in such a way that user code wouldn’t notice.
>
> This stack splitting code is where the name “nosplit” comes from.

A nosplit function does not load and branch on `runtime.g.stackguard0`, and simply assumes it has enough stack. This means that nosplit functions will not preempt themselves, and, as a result, are noticeably faster to call in a hot loop. Don’t believe me?

```go highlight
//go:noinline
func noinline(x int) {}

//go:nosplit
func nosplit(x int) { noinline(x) }
func yessplit(x int) { noinline(x) }

func BenchmarkCall(b *testing.B) {
  b.Run("nosplit", func(b *testing.B) {
    for b.Loop() { nosplit(42) }
  })
  b.Run("yessplit", func(b *testing.B) {
    for b.Loop() { yessplit(42) }
  })
}

```

[Go](#code:2)

If we profile this and pull up the timings for each function, here’s what we get:

```x86-go highlight
390ms      390ms           func nosplit(x int) { noinline(x) }
 60ms       60ms   51fd80:     PUSHQ BP
 10ms       10ms   51fd81:     MOVQ SP, BP
    .          .   51fd84:     SUBQ $0x8, SP
 60ms       60ms   51fd88:     CALL .noinline(SB)
190ms      190ms   51fd8d:     ADDQ $0x8, SP
    .          .   51fd91:     POPQ BP
 70ms       70ms   51fd92:     RET

440ms      490ms           func yessplit(x int) { noinline(x) }
 50ms       50ms   51fda0:     CMPQ SP, 0x10(R14)
 20ms       20ms   51fda4:     JBE 0x51fdb9
    .          .   51fda6:     PUSHQ BP
 20ms       20ms   51fda7:     MOVQ SP, BP
    .          .   51fdaa:     SUBQ $0x8, SP
 10ms       60ms   51fdae:     CALL .noinline(SB)
200ms      200ms   51fdb3:     ADDQ $0x8, SP
    .          .   51fdb7:     POPQ BP
140ms      140ms   51fdb8:     RET
    .          .   51fdb9:     MOVQ AX, 0x8(SP)
    .          .   51fdbe:     NOPW
    .          .   51fdc0:     CALL runtime.morestack_noctxt.abi0(SB)
    .          .   51fdc5:     MOVQ 0x8(SP), AX
    .          .   51fdca:     JMP .yessplit(SB)

```

[x86 Assembly (Go Syntax)](#code:3)

The time spent at each instruction (for the whole benchmark, where I made sure equal time was spent on each test case with `-benchtime Nx`) is comparable for all of the instructions these functions share, but an additional ~2% cost is incurred for the stack check.

This is a very artificial setup, because the `g` struct is always in L1 in the `yessplit` benchmark due to the fact that no other memory operations occur in the loop. However, for very hot code that needs to saturate the cache, this can have an outsized effect due to cache misses. We can enhance this benchmark by adding an assembly function that executes `clflush [r14]`, which causes the `g` struct to be ejected from all caches.

```x86-go highlight
  TEXT .clflush(SB)
  CLFLUSH (R14)  // Eject the pointee of r14 from all caches.
  RET

```

[x86 Assembly (Go Syntax)](#code:4)

If we add a call to this function to both benchmark loops, we see the staggering cost of a cold fetch from RAM show up in every function call: 120.1 nanosecods for `BenchmarkCall/nosplit`, versus 332.1 nanoseconds for `BenchmarkCall/yessplit`. The 200 nanosecond difference is a fetch from main memory. An L1 miss is about 15 times less expensive, so if the `g` struct manages to get kicked out of L1, you’re paying about 15 or so nanoseconds, or about two map lookups!

Despite the language resisting adding an inlining heuristic, which programmers would place everywhere without knowing what it does, they *did* provide something worse that makes code noticeably faster: nosplit.

## [But It’s Harmless…?](\#but-its-harmless)

Consider the following program[2](#fn:2):

```go highlight
//go:nosplit
func x(y int) { x(y+1) }

```

[Go](#code:5)

Naturally, this will instantly overflow the stack. Instead, we get a really scary linker error:

```console highlight
x.x: nosplit stack over 792 byte limit
x.x<1>
    grows 24 bytes, calls x.x<1>
    infinite cycle

```

[Console](#code:6)

The Go linker contains a check to verify that any chain of nosplit functions which call nosplit functions do not overflow a small window of extra stack, which is where the stack frames of nosplit functions live if they go past `stackguard0`.

Every stack frame contributes some stack use (for the return address, at minimum), so the number of functions you can call before you get this error is limited. And because every function needs to allocate space for all of its callees to spill their arguments if necessary, you can hit this limit every fast if every one of these functions uses every available argument register (ask me how I know).

Also, turning on fuzzing instruments the code by inserting nosplit calls into the fuzzer runtime around branches, meaning that turning on fuzzing can *previously fine code to no longer link*. Stack usage also varies slightly by architecture, meaning that code which builds in one architecture fails to link in others (most visible when going from 32-bit to 64-bit).

There is no easy way to control directives using build tags (two poorly-designed features collide), so you cannot just “turn off” performance-sensitive nosplits for debugging, either.

For this reason, you must be *very very careful* about using nosplit for performance.

### [Virtual Nosplit Functions](\#virtual-nosplit-functions)

Excitingly, nosplit functions whose addresses are taken do not have special codegen, allowing us to defeat the linker stack check by using virtual function calls.

Consider the following program:

```go highlight
package main

var f func(int)

//go:nosplit
func x(y int) { f(y+1) }

func main() {
  f = x
  f(0)
}

```

[Go](#code:7)

This will quickly exhaust the main G’s tiny stack and segfault in the most violent way imaginable, preventing the runtime from printing a debug trace. All this program outputs is `signal: segmentation fault`.

This is [probably a bug](https://github.com/golang/go/issues/74478).

## [Other Side Effects](\#other-side-effects)

It turns out that nosplit has various other fun side-effects that are not documented anywhere. The main thing it does is it contributes to whether a function is considered “unsafe” by the runtime.

Consider the following program:

```go highlight
package main

import (
  "fmt"
  "os"
  "runtime"
  "time"
)

func main() {
  for range runtime.GOMAXPROCS(0) {
    go func() {
      for {}
    }()
  }
  time.Sleep(time.Second) // Wait for all the other Gs to start.

  fmt.Println("Hello, world!")
  os.Exit(0)
}

```

[Go](#code:8)

This program will make sure that every P becomes bound to a G that loops forever, meaning they will never trap into the runtime. Thus, this program will hang forever, never printing its result and exiting. But that’s not what happens.

Thanks to asynchronous preemption, the scheduler will detect Gs that have been running for too long, and preempt its M by sending a signal to it ( [due to happenstance](https://cs.opensource.google/go/go/+/master:src/runtime/signal_unix.go;l=43), this is `SIGURG` of all things.)

However, asynchronous preemption is only possible when the M stops due to the signal at a safe point, as determined by `runtime.isAsyncSafePoint`. It includes the following block of code:

```go highlight
	up, startpc := pcdatavalue2(f, abi.PCDATA_UnsafePoint, pc)
	if up == abi.UnsafePointUnsafe {
		// Unsafe-point marked by compiler. This includes
		// atomic sequences (e.g., write barrier) and nosplit
		// functions (except at calls).
		return false, 0
	}

```

[Go](#code:9)

If we chase down where this value is set, we’ll find that it is set explicitly for write barrier sequences, for any function that is “part of the runtime” (as defined by being built with the `-+` flag) and *for any nosplit function*.

With a small modification of hoisting the `go` body into a nosplit function, the following program will run forever: it will never wake up from `time.Sleep`.

```go highlight
package main

import (
  "fmt"
  "os"
  "runtime"
  "time"
)

//go:nosplit
func forever() {
  for {}
}

func main() {
  for range runtime.GOMAXPROCS(0) {
    go forever()
  }
  time.Sleep(time.Second) // Wait for all the other Gs to start.

  fmt.Println("Hello, world!")
  os.Exit(0)
}

```

[Go](#code:10)

Even though there is work to do, every P is bound to a G that will never reach a safe point, so there will never be a P available to run the main goroutine.

This represents another potential danger of using nosplit functions: those that do not call preemptable functions must terminate promptly, or risk livelocking the whole runtime.

## [Conclusion](\#conclusion)

I use nosplit *a lot*, because I write high-performance, low-latency Go. This is a very insane thing to do, which has caused me to slowly generate bug reports whenever I hit strange corner cases.

For example, there are many cases where spill regions are allocated for functions that never use them, for example, functions which only call nosplit functions allocate space for them to spill their arguments, which they don’t do.[3](#fn:3)

This is a documented Go language feature which:

1. Isn’t very well-documented (the async preemption behavior certainly isn’t)!
2. Has very scary optimization-dependent build failures.
3. Can cause livelock and mysterious segfaults.
4. Can be used in user programs that don’t `import "unsafe"`!
5. And it makes code faster!

I’m surprised such a massive footgun exists at all, buuuut it’s a measureable benchmark improvement for me, so it’s impossible to tell if it’s bad or not.

---

1. The astute reader will observe that because preemption is not reentrant, only one of these spill regions will be in use at at time in a G. This is a known bug in the ABI, and is essentially a bodge to enable easy adoption of passing arguments by register, without needing all of the parts of the runtime that expect arguments to be spilled to the stack, as was the case in the slow old days when Go’s ABI on every platform was “ `i386-unknown-linux` but worse”, i.e., arguments went on the stack and made the CPU’s store queue sad.

I recently filed [a bug about this](https://github.com/golang/go/issues/74413) that boils down to “add a field to `runtime.g` to use a spill space”, which seems to me to be simpler than the alternatives described in the `ABIInternal` spec. [↩︎](#fnref:1)

2. Basically every bug report I write starts with these four words and it means you’re about to see the worst program ever written. [↩︎](#fnref:2)

3. The spill area is also used for spilling arguments across calls, but in this case, it is not necessary for the caller to allocate it for a nosplit function. [↩︎](#fnref:3)
