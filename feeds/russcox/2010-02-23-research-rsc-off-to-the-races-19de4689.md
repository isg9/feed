---
title: 'research!rsc: Off to the Races'
url: https://research.swtch.com/gorace
published: "2010-02-23T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/gorace
---

# research!rsc: Off to the Races

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Off to the Races Russ Cox February 23, 2010 *research.swtch.com/gorace* Posted on Tuesday, February 23, 2010.

Go is defined to be a safe language. Indices into array or string references must be in bounds; there is no way to reinterpret the bits of one type as another, no way to conjure a pointer out of thin air; and there is no way to release memory, so no chance of “ [dangling pointer](http://en.wikipedia.org/wiki/Dangling_pointer)” errors and the associated memory corruption and instability.

In the current Go implementations, though, there are two ways to break through these safety mechanisms. The first and more direct way is to use [package unsafe](http://golang.org/pkg/unsafe/), specifically [unsafe.Pointer](http://golang.org/pkg/unsafe#Pointer). The second, less direct way is to use a data race in a multithreaded program.

If you were going to build an environment that ran untrusted Go code, you'd probably want to change the available packages to restrict or delete certain routines, like [os.RemoveAll](http://golang.org/pkg/os/#RemoveAll), and you'd want to disallow access to package unsafe. Those kinds of restrictions are straightforward.

The data races that can be used to break through the usual memory safety of Go are less straightforward. This post describes the races and how to rearrange the data structures involved to avoid them. Until the Go implementations have been tuned more, we won't be able to measure whether there is a significant performance difference between the current representation and the race-free implementation.

### Package Unsafe

Here's a simple packaging of a type that lets you edit arbitrary memory locations, built using the standard [package unsafe](http://golang.org/pkg/unsafe/):

```
import "unsafe"

type Mem struct {
 addr *uintptr // actually == &m.data!
 data *uintptr
}

// Peek reads and returns the word at address addr.
func (m *Mem) Peek(addr uintptr) uintptr {
 *m.addr = addr
 return *m.data
}

// Poke sets the word at address addr to val.
func (m *Mem) Poke(addr, val uintptr) {
 *m.addr = addr
 *m.data = val
}

func NewMem() *Mem {
 m := new(Mem)
 m.addr = (*uintptr)(unsafe.Pointer(&m.data))
 return m
}

```

(The Go type `uintptr` is an unsigned integer the size of a pointer, like `uint32` or `uint64` depending on the underlying machine architecture.)

The key line is near the bottom, the use of the special type `unsafe.Pointer` to convert a `**uintptr` into a `*uintptr`. Dereferencing that value gives us `m.data` (actually a `*uintptr`) interpreted as a `uintptr`. We can assign an arbitrary integer to `*m.addr`, that changes `m.data`, and then we can dereference the integer as `*m.data`. In other words, the `Mem` struct gives us a way to convert between integers and pointers, just like in C. There are no races here: this is just something you can do by importing `unsafe`. The `Mem` wrapper is a bit convoluted—normally you'd just use `unsafe` directly—but we're going to drop in a different implementation of `NewMem` that doesn't rely on `unsafe`.

### A Race

The current Go representation of slices and interface values admits a [data race](http://en.wikipedia.org/wiki/Race_condition): because they are multiword values, if one goroutine reads the value while another goroutine writes it, the reader might see half of the old value and half of the new value.

Let's provoke the race using interface values. In Go, an interface value is [represented as two words](http://research.swtch.com/2009/12/go-data-structures-interfaces.html), a type and a value of that type. After these declarations:

```
var x *uintptr

var i interface{} = (*uintptr)(nil)
var j interface{} = &x

```

The data structures for `i` and `j` look like:

![](https://research.swtch.com/gorace1.png)

Suppose we kick off a goroutine that alternately assigns `i` and `j` to a new interface value `k`:

```
var k interface{}

func hammer() {
 for {
  k = i
  k = j
 }
}

```

After each statement executes, `k` will look like either `i` or `j`, but during the assignment, there will be a moment when `k` is half `i` and half `j`, one of these:

![](https://research.swtch.com/gorace2.png)

The top case gives us a `**uintptr` `nil`, which we could obtain more easily via legitimate means, but the bottom case gives us the value `&x` (actually a `**uintptr`) interpreted as a `*uintptr`. If we can catch the interface when it looks like the case on the right, we'll have rederived the conversion we used above via `unsafe`. Based on that insight, we can rewrite `NewMem` without `unsafe`:

```
func NewMem() *Mem {
 fmt.Println("here we go!")

 m := new(Mem)

 var i, j, k interface{}
 i = (*uintptr)(nil)
 j = &m.data

 // Try over and over again until we win the race.
 done := false
 go func(){
  for !done {
   k = i
   k = j
  }
 }()
 for {
  // Is k a non-nil *uintptr?  If so, we got it.
  if p, ok := k.(*uintptr); ok && p != nil {
   m.addr = p
   done = true
   break
  }
 }
 return m
}

```

The same kind of race happens in all of Go's mutable multiword structures: slices, interfaces, and strings. In the case of slices, the trick is to get a pointer from one slice and a cap from a different one. In the case of strings, the trick is to get a pointer from one string and the len from a different one. (The string race isn't as interesting, because strings cannot be written to, so it would only let you read memory, not write it.)

### The Fix

The race is fundamentally caused by being able to observe partial updates to Go's multiword values (slices, interfaces, and strings): the updates are not atomic.

The fix is to make the updates atomic. In Go, the easiest way to do that is to make the representation a single pointer that points at an immutable structure. When the value needs to be updated, you allocate a new structure, fill it in completely, and only then change the pointer to point at it. This makes the assignment atomic: another goroutine reading the pointer at the same time sees either the new data or the old data, but not a mix, assuming the compiler is careful to read the pointer just once and then access both fields using the same pointer value.

![](https://research.swtch.com/gorace3.png)

(The red border indicates immutable data.)

For slices and strings, it makes sense to keep the multiword representation but put an immutable ”pointer and cap” stub structure between the slice and the underlying array. This keeps the same basic efficiency properties of slices at the cost of a few extra instructions on each indexing operation.

![](https://research.swtch.com/gorace4.png)

The idea here is to keep a structure with a mutable offset and length to support efficient slicing but replace the pointer with an immutable base+length pair. Any access to the underlying data must check the final offset against the immutable *cap*. Copying slice values is still not an atomic operation, but an invalid *len* will not keep an out-of-bounds index from being caught.

This representation requires a couple more assembly instructions, because each index must be checked against two bounds, first the relative len and then the absolute cap:

Compute `x[i]` in `AX`*Racy__Race-free*

```
LEAL x, SI
MOVL i, CX
CMPL CX, 4(SI)
JGE panic

MOVL 0(SI), DI
MOVL (4*CX)(DI), AX

```

```
LEAL x, BX
MOVL i, CX
CMPL CX, 4(BX)
JGE panic
ADDL 8(BX), CX
MOVL 0(BX), SI
CMPL CX, 4(SI)
JGE panic
MOVL 0(SI), DI
MOVL (4*CX)(DI), AX

```

```


// i >= len?


// i+off >= cap?

// &x[0] -> SI
// x[i] (or x[i+off]) -> DI


```

With suitable analysis, an optimizing compiler could cache `0(BX)`, `4(BX)`, `8(BX)`, and `4(SI)`, so in a loop, it is possible that the new representation would run at the same speed as the original.

An ambitious implementation might continue to use the current data structures for slices, interfaces, and strings stored on the stack, because data on the stack can only be accessed by the goroutine running on that stack. (Local variables whose addresses might escape to other goroutines are already allocated on the heap automatically, to avoid dangling pointer bugs after a function returns.)

### Garbage Collection

This fix is feasible only because Go is a garbage-collected language: we can treat the red stub structures as immutable and trust that the garbage collector will recycle the memory only when nothing points to them anymore. It's much harder to build a safe language without a garbage collector to fall back on.

### Security Implications

It is important to note that these races do not make the current implementations any less secure than they already are. The races allow clever programmers to subvert Go's memory safety, but a less clever programmer can still use the aptly-named [package unsafe](http://golang.org/doc/go_spec.html#Package_unsafe).

These races only matter if you are trying to build a Go service that can safely run arbitrary code supplied by untrusted programmers (and to the best of my knowledge, there are no such services yet). In that situation, you'd already need to change the implementations to disable access to the unsafe package and remove or restrict functions like os.Remove or net.Dial. Changing the data representations to be race free is just one more change you'd have to make. Now you know, not just that a change is needed but also what the change is.

The races exist because the data representations were chosen for performance: the race-free versions introduce an extra pointer, which carries with it the cost of extra indirection and extra allocation. Once the Go implementations are more mature, we'll be able to evaluate the precise performance impact of using the race-free data structures and whether to use them always or only in situations running untrusted code.

(Comments originally posted via Blogger.)

- [Ed Marshall](http://www.blogger.com/profile/18124558793142592579)(February 23, 2010 10:28 AM) *These races only matter if you are trying to build a Go service that can safely run arbitrary code supplied by untrusted programmers (and to the best of my knowledge, there are no such services yet).*

\*cough\* [AppEngine](http://code.google.com/appengine/)\*cough\*? :)

- [stefanha](http://www.blogger.com/profile/00351576818248952080)(February 23, 2010 2:26 PM) In your example, is there any guarantee that setting 'done' to true will ever be visible to the anonymous goroutine?

(Would a compiler be allowed to assume that since there is no synchronization point inside the loop and 'done' is not modified in the loop, it does not need to load 'done' from memory each loop iteration?)

Also, this race seems like an issue for legitimate programs. If interface{}, slice, and string assignment are not atomic, then robust code accessing such variables concurrently needs to explicitly synchronize in order to work around the current non-atomic implementation. :(

- [Russ Cox](http://swtch.com/~rsc/)(February 23, 2010 4:11 PM) @stefanha: No, there's no guarantee in the memory model that the use of done is okay. But I happen to know it works in the current implementation. ;-)

Regarding the memory model, interface{}, slice, and string updates are not guaranteed to be atomic (in fact, no variable access is guaranteed to be atomic), so a correct program **should** be synchronizing. That's a good thing.

- [metageek](http://metageek.livejournal.com/)(March 7, 2010 5:23 AM) Wouldn't it be expensive to require extra memory allocations for all the interfaces? Instead, the compiler could emit double-word atomic operations like CMPXCHG8B.

- [Jessta](http://www.blogger.com/profile/06837651109419168637)(May 19, 2010 2:51 PM) I found a tiny typo,

The text says

var i interface{} = &x

var j interface{} = (\*uintptr)(nil)

but the diagrams say,

var i interface{} = (\*uintptr)(nil)

var j interface{} = &x

- [Santiago Gala](http://www.blogger.com/profile/00397201887944230084)(October 16, 2010 7:26 AM) @rsc (regarding done): isn't done heap allocated in the heap (as it is used from the (potentially long lived) goroutine outside of the function where it is declared, and aren't those kind of heap accessed treated as "volatile"?

- [Russ Cox](http://swtch.com/~rsc/)(November 9, 2010 7:36 AM) @santiago: Done is in the heap, but Go has no concept of "volatile". The compiler might well optimize the accesses to done in the tight loop since it can see that done is not changing. But I know it doesn't.

Go guarantees almost nothing about memory accesses unless you explicitly synchronize with locks or channels. http://golang.org/doc/go\_mem.html

- [mk](http://www.blogger.com/profile/18233398123137441204)(November 25, 2010 3:08 PM) Good article, but I think Jessta is correct...the first picture in the Race section looks flat-out wrong.

Could you fix the picture or switch i/j in the code immediately preceding it?

- [Russ Cox](http://swtch.com/~rsc/)(December 4, 2010 8:41 AM) @mk, @jessta: Fixed the text, thanks.

- Anonymous (May 11, 2011 10:22 AM) Google recently announced Go support on the App Engine service. Does this vulnerability still exist?

- [Russ Cox](http://swtch.com/~rsc/)(May 23, 2011 6:40 AM) The race described in this post only exists in multithreaded programs. Go on App Engine is restricted to a single thread (running possibly many goroutines).

- [jam](http://www.blogger.com/profile/17344213294371886790)(July 21, 2011 4:08 AM) This is also hidden behind the language specification, right? So if Google wanted to have their AppEngine compiler emit the 8-byte compare&exchange, or switch to the pointer indirection, it would still run the go code everyone else was running.

It is also trivial to rewrite the "done" logic with a non-blocking channel (either via select, or via val, ok, I believe.)

I added some counters for how many steps it takes. And certainly GOMAXPROCS=1 it never happens. Otherwise I've seen it as fast as:

Got it after 2567 attempts, 19 go\_steps

and as slow as:

Got it after 302132 attempts, 87105 go\_steps

The latter was using a channel synchronization, though, which might make things more 'ordered'.

- [⚛](http://www.blogger.com/profile/00398184141815003668)(October 4, 2011 2:51 AM) http://stackoverflow.com/questions/7646018/sse-instructions-single-memory-access

- [Russ Cox](http://swtch.com/~rsc/)(October 30, 2011 9:01 AM) ⚛, very cool, thanks.
