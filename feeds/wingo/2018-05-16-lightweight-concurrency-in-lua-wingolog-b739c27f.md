---
title: lightweight concurrency in lua — wingolog
url: https://wingolog.org/archives/2018/05/16/lightweight-concurrency-in-lua
published: "2018-05-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/05/16/lightweight-concurrency-in-lua
---

# lightweight concurrency in lua — wingolog

## [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)

16 May 2018 3:17 PM

- [concurrency](/tags/concurrency)
- [lua](/tags/lua)
- [fibers](/tags/fibers)
- [guile](/tags/guile)
- [concurrent ml](/tags/concurrent%20ml)
- [coroutines](/tags/coroutines)
- [delimited continuations](/tags/delimited%20continuations)
- [igalia](/tags/igalia)
- [snabb](/tags/snabb)

Hello, all! Today I'd like to share some work I have done recently as part of the [Snabb user-space networking toolkit](https://github.com/snabbco/snabb). Snabb is mainly about high-performance packet processing, but it also needs to communicate with management-oriented parts of network infrastructure. These communication needs are performed by a dedicated [manager process](https://github.com/snabbco/snabb/blob/master/src/lib/ptree/README.md), but that process has many things to do, and can't afford to make blocking operations.

Snabb is written in Lua, which doesn't have built-in facilities for concurrency. What we'd like is to have [fibers](https://wingolog.org/archives/2017/06/27/growing-fibers). Fortunately, Lua's coroutines are powerful enough to implement fibers. Let's do that!

**fibers in lua**

First we need a scheduling facility. Here's the smallest possible scheduler: simply a queue of tasks and a function to run those tasks.

```
local task_queue = {}

function schedule_task(thunk)
   table.insert(task_queue, thunk)
end

function run_tasks()
   local queue = task_queue
   task_queue = {}
   for _,thunk in ipairs(queue) do thunk() end
end

```

For our purposes, a task is just a function that will be called with no arguments.

Now let's build fibers. This is easier than you might think!

```
local current_fiber = false

function spawn_fiber(fn)
   local fiber = coroutine.create(fn)
   schedule_task(function () resume_fiber(fiber) end)
end

function resume_fiber(fiber, ...)
   current_fiber = fiber
   local ok, err = coroutine.resume(fiber, ...)
   current_fiber = nil
   if not ok then
      print('Error while running fiber: '..tostring(err))
   end
end

function suspend_current_fiber(block, ...)
   -- The block function should arrange to reschedule
   -- the fiber when it becomes runnable.
   block(current_fiber, ...)
   return coroutine.yield()
end

```

Here, a fiber is simply a coroutine underneath. Suspending a fiber suspends the coroutine. Resuming a fiber runs the coroutine. If you're unfamiliar with coroutines, or coroutines in Lua, maybe have a look at [the lua-users wiki page on the topic](http://lua-users.org/wiki/CoroutinesTutorial).

The difference between a fibers facility and just coroutines is that with fibers, you have a scheduler as well. Very much like Scheme's [`call-with-prompt`](https://www.gnu.org/software/guile/manual/html_node/Prompt-Primitives.html), coroutines are one of those powerful language building blocks that should rarely be used directly; concurrent programming needs more structure than what Lua offers.

If you're following along, it's probably worth it here to think how you would implement `yield` based on these functions. A `yield` implementation should yield control to the scheduler, and resume the fiber on the next scheduler turn. The answer is [here](https://gitlab.com/snippets/1715966#L34).

**communication**

Once you have fibers and a scheduler, you have concurrency, which means that if you're not careful, you have a mess. Here I think the Go language got the essence of the idea exactly right: [Do not communicate by sharing memory; instead, share memory by communicating.](https://blog.golang.org/share-memory-by-communicating)

Even though Lua doesn't support multiple machine threads running concurrently, concurrency between fibers can still be fraught with bugs. Tony Hoare's [Communicating Sequential Processes](http://usingcsp.com/) showed that we can avoid a class of these bugs by treating communication as a first-class concept.

Happily, the Concurrent ML project showed that it's possible to build these first-class communication facilities as a library, provided the language you are working in has threads of some kind, and fibers are enough. Last year I built a [Concurrent ML library for Guile Scheme](https://wingolog.org/archives/2017/06/29/a-new-concurrent-ml), and when in [Snabb](https://github.com/snabbco/snabb) we had a similar need, I ported that code over to Lua. As it's a new take on the problem in a different language, I think I've been able to simplify things even more.

So let's take a crack at implementing Concurrent ML in Lua. In CML, the fundamental primitive for communication is the *operation*. An operation represents the potential for communication. For example, if you have a channel, it would have methods to return "get operations" and "put operations" on that channel. Actually receiving or sending a message on a channel occurs by *performing* those operations. One operation can be performed many times, or not at all.

Compared to a system like Go, for example, there are two main advantages of CML. The first is that CML allows non-deterministic choice between a number of potential operations in a generic way. For example, you can construct a operation that, when performed, will either get on one channel or wait for a condition variable to be signalled, whichever comes first. In Go, you can only `select` between operations on channels.

The other interesting part of CML is that operations are built from a uniform protocol, and so users can implement new kinds of operations. Compare again to Go where all you have are channels, and nothing else.

The CML operation protocol consists three related functions: `try` which attempts to directly complete an operation in a non-blocking way; `block`, which is called after a fiber has suspended, and which arranges to resume the fiber when the operation completes; and `wrap`, which is called on the result of a successfully performed operation.

In Lua, we can call this an *implementation* of an operation, and create it like this:

```
function new_op_impl(try, block, wrap)
   return { try=try, block=block, wrap=wrap }
end

```

Now let's go ahead and write the guts of CML: the operation implementation. We'll represent an operation as a Lua object with two methods. The `perform` method will attempt to perform the operation, and return the resulting value. If the operation can complete immediately, the call to `perform` will return directly. Otherwise, `perform` will suspend the current fiber and arrange to continue only when the operation completes.

The `wrap` method "decorates" an operation, returning a new operation that, if and when it completes, will "wrap" the result of the completed operation with a function, by applying the function to the result. It's useful to distinguish the sub-operations of a non-deterministic choice from each other.

Here our `new_op` function will take an array of operation implementations and return an operation that, when performed, will synchronize on the first available operation. As you can see, it already has the equivalent of Go's `select` built in.

```
function new_op(impls)
   local op = { impls=impls }

   function op.perform()
      for _,impl in ipairs(impls) do
         local success, val = impl.try()
         if success then return impl.wrap(val) end
      end
      local function block(fiber)
         local suspension = new_suspension(fiber)
         for _,impl in ipairs(impls) do
            impl.block(suspension, impl.wrap)
         end
      end
      local wrap, val = suspend_current_fiber(block)
      return wrap(val)
   end

   function op.wrap(f)
      local wrapped = {}
      for _, impl in ipairs(impls) do
         local function wrap(val)
            return f(impl.wrap(val))
         end
         local impl = new_op_impl(impl.try, impl.block, wrap)
         table.insert(wrapped, impl)
      end
      return new_op(wrapped)
   end

   return op
end

```

There's only one thing missing there, which is `new_suspension`. When you go to suspend a fiber because none of the operations that it's trying to do can complete directly (i.e. all of the `try` functions of its impls returned false), at that point the corresponding `block` functions will publish the fact that the fiber is waiting. However the fiber only waits until the first operation is ready; subsequent operations becoming ready should be ignored. The suspension is the object that manages this state.

```
function new_suspension(fiber)
   local waiting = true
   local suspension = {}
   function suspension.waiting() return waiting end
   function suspension.complete(wrap, val)
      assert(waiting)
      waiting = false
      local function resume()
         resume_fiber(fiber, wrap, val)
      end
      schedule_task(resume)
   end
   return suspension
end

```

As you can see, the suspension's `complete` method is also the bit that actually arranges to resume a suspended fiber.

Finally, just to round out the implementation, here's a function implementing non-deterministic choice from among a number of sub-operations:

```
function choice(...)
   local impls = {}
   for _, op in ipairs({...}) do
      for _, impl in ipairs(op.impls) do
         table.insert(impls, impl)
      end
   end
   return new_op(impls)
end

```

**on cml**

OK, I'm sure this seems a bit abstract at this point. Let's implement something concrete in terms of these primitives: channels.

Channels expose two similar but different kinds of operations: put operations, which try to send a value, and get operations, which try to receive a value. If there's a sender already waiting to send when we go to perform a `get_op`, the operation continues directly, and we resume the sender; otherwise the receiver publishes its suspension to a queue. The `put_op` case is similar.

Finally we add some synchronous `put` and `get` convenience methods, in terms of their corresponding CML operations.

```
function new_channel()
   local ch = {}
   -- Queues of suspended fibers waiting to get or put values
   -- via this channel.
   local getq, putq = {}, {}

   local function default_wrap(val) return val end
   local function is_empty(q) return #q == 0 end
   local function peek_front(q) return q[1] end
   local function pop_front(q) return table.remove(q, 1) end
   local function push_back(q, x) q[#q+1] = x end

   -- Since a suspension could complete in multiple ways
   -- because of non-deterministic choice, it could be that
   -- suspensions on a channel's putq or getq are already
   -- completed.  This helper removes already-completed
   -- suspensions.
   local function remove_stale_entries(q)
      local i = 1
      while i <= #q do
         if q[i].suspension.waiting() then
            i = i + 1
         else
            table.remove(q, i)
         end
      end
   end

   -- Make an operation that if and when it completes will
   -- rendezvous with a receiver fiber to send VAL over the
   -- channel.  Result of performing operation is nil.
   function ch.put_op(val)
      local function try()
         remove_stale_entries(getq)
         if is_empty(getq) then
            return false, nil
         else
            local remote = pop_front(getq)
            remote.suspension.complete(remote.wrap, val)
            return true, nil
         end
      end
      local function block(suspension, wrap)
         remove_stale_entries(putq)
         push_back(putq, {suspension=suspension, wrap=wrap, val=val})
      end
      return new_op({new_op_impl(try, block, default_wrap)})
   end

   -- Make an operation that if and when it completes will
   -- rendezvous with a sender fiber to receive one value from
   -- the channel.  Result is the value received.
   function ch.get_op()
      local function try()
         remove_stale_entries(putq)
         if is_empty(putq) then
            return false, nil
         else
            local remote = pop_front(putq)
            remote.suspension.complete(remote.wrap)
            return true, remote.val
         end
      end
      local function block(suspension, wrap)
         remove_stale_entries(getq)
         push_back(getq, {suspension=suspension, wrap=wrap})
      end
      return new_op({new_op_impl(try, block, default_wrap)})
   end

   function ch.put(val) return ch.put_op(val).perform() end
   function ch.get()    return ch.get_op().perform()    end

   return ch
end

```

**a wee example**

You might be wondering what it's like to program with channels in Lua, so here's a little example that shows a prime sieve based on channels. It's not a great example of concurrency in that it's not an inherently concurrent problem, but it's cute to show computations in terms of infinite streams.

```
function prime_sieve(count)
   local function sieve(p, rx)
      local tx = new_channel()
      spawn_fiber(function ()
         while true do
            local n = rx.get()
            if n % p ~= 0 then tx.put(n) end
         end
      end)
      return tx
   end

   local function integers_from(n)
      local tx = new_channel()
      spawn_fiber(function ()
         while true do
            tx.put(n)
            n = n + 1
         end
      end)
      return tx
   end

   local function primes()
      local tx = new_channel()
      spawn_fiber(function ()
         local rx = integers_from(2)
         while true do
            local p = rx.get()
            tx.put(p)
            rx = sieve(p, rx)
         end
      end)
      return tx
   end

   local done = false
   spawn_fiber(function()
      local rx = primes()
      for i=1,count do print(rx.get()) end
      done = true
   end)

   while not done do run_tasks() end
end

```

Here you also see an example of running the scheduler in the last line.

**where next?**

Let's put this into perspective: in a couple hundred lines of code, we've gone from minimal Lua to a language with lightweight multitasking, extensible CML-based operations, and CSP-style channels; truly a delight.

There are a number of possible ways to extend this code. One of them is to implement true multithreading, if the language you are working in supports that. In that case there are some small protocol modifications to take into account; see [the notes on the Guile CML implementation](https://wingolog.org/archives/2017/06/29/a-new-concurrent-ml) and especially the [Manticore Parallel CML](http://manticore.cs.uchicago.edu/) project.

The implementation above is pleasantly small, but it could be faster with the choice of more specialized data structures. I think interested readers probably see a number of opportunities there.

In a library, you might want to avoid the global `task_queue` and implement nested or multiple independent schedulers, and of course in a parallel situation you'll want core-local schedulers as well.

The implementation above has no notion of time. What we did in the [Snabb implementation of fibers](https://github.com/Igalia/snabb/tree/lwaftr/src/lib/fibers) was to implement a [timer wheel](https://github.com/Igalia/snabb/blob/lwaftr/src/lib/fibers/timer.lua), inspired by [Juho Snellman's Ratas](https://www.snellman.net/blog/archive/2016-07-27-ratas-hierarchical-timer-wheel/), and then add that timer wheel as a task source to Snabb's scheduler. In Snabb, every time the equivalent of `run_tasks()` is called, a scheduler asks its sources to schedule additional tasks. The timer wheel implementation schedules expired timers. It's straightforward to build [CML timeout operations](https://github.com/Igalia/snabb/blob/lwaftr/src/lib/fibers/sleep.lua) in terms of timers.

Additionally, your system probably has other external sources of communication, such as sockets. The trick to integrating sockets into fibers is to suspend the current fiber whenever an operation on a file descriptor would block, and arrange to resume it when the operation can proceed. [Here's the implementation in Snabb](https://github.com/Igalia/snabb/blob/lwaftr/src/lib/fibers/file.lua).

The only difficult bit with getting nice nonblocking socket support is that you need to be able to suspend the calling thread when you see the `EWOULDBLOCK` condition, and for coroutines that is often only possible if you implemented the buffered I/O yourself. In Snabb that's what we did: we implemented [a compatible replacement for Lua's built-in streams, in Lua](https://github.com/Igalia/snabb/blob/lwaftr/src/lib/fibers/file.lua). That lets us handle `EWOULDBLOCK` conditions in a flexible manner. Integrating `epoll` as a task source also lets us sleep when there are no runnable tasks.

Likewise in the Snabb context, we are also working on a TCP implementation. In that case you want to structure TCP endpoints as fibers, and arrange to suspend and resume them as appropriate, while also allowing timeouts. I think the scheduler and CML patterns are going to allow us to do that without much trouble. (Of course, the TCP implementation will give us lots of trouble!)

Additionally your system might want to communicate with fibers from other threads. It's entirely possible to implement CML on top of pthreads, and it's entirely possible as well to support communication between pthreads and fibers. If this is interesting to you, see Guile's implementation.

When I talked about fibers in [an earlier article](https://wingolog.org/archives/2017/06/27/growing-fibers), I built them in terms of delimited continuations. Delimited continuations are fun and more expressive than coroutines, but it turns out that for fibers, all you need is the expressive power of coroutines -- multi-shot continuations aren't useful. Also I think the presentation might be more straightforward. So if all your language has is coroutines, that's still good enough.

There are many more kinds of standard CML operations; implementing those is also another next step. In particular, I have found semaphores and condition variables to be quite useful. Also, standard CML supports "guards", invoked when an operation is performed, and "nacks", invoked when an operation is definitively *not* performed because a `choice` selected some other operation. These can be layered on top; see the Parallel CML paper for notes on "primitive CML".

Also, the `choice` operator above is left-biased: it will prefer earlier impls over later ones. You might want to not always start with the first impl in the list.

The scheduler shown above is the simplest thing I could come up with. You may want to experiment with other scheduling algorithms, e.g. [capability-based scheduling](https://dl.acm.org/citation.cfm?doid=3190508.3190539), or [kill-safe abstractions](https://www.cs.utah.edu/plt/publications/pldi04-ff.pdf). Do it!

Or, it could be you already have a scheduler, like some kind of main loop that's already there. Cool, you can use it directly -- all that fibers needs is some way to schedule functions to run.

**godspeed**

In summary, I think Concurrent ML should be better-known. Its simplicity and expressivity make it a valuable part of any concurrent system. Already in Snabb it helped us solve some [longstanding gnarly issues](https://github.com/Igalia/snabb/issues/668) by making the right solutions expressible.

As Adam Solove says, [Concurrent ML is great, but it has a branding problem](https://medium.com/@asolove/concurrent-ml-has-a-branding-problem-ce0286eab598). Its ideas haven't penetrated the industrial concurrent programming world to the extent that they should. This article is another attempt to try to get the word out. Thanks to Adam for the observation that CML is really a protocol; I'm sure the concepts could be made even more clear, but at least this is a step forward.

All the code in this article is up on a [gitlab snippet](https://gitlab.com/snippets/1715966) along with instructions for running the example program from the command line. Give it a go, and happy hacking with CML!

## related articles

- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)
- [unintentional concurrency](/archives/2022/07/20/unintentional-concurrency)
- [An incomplete history of language facilities for concurrency](/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency)

### One response

1. [Arne Babenhauserheide](http://draketo.de) says:[16 July 2018 10:04 PM](#0920a20d997ac3f351eec1283b20b3d39faba9e3)

   Do you have a performance benchmark of the lua-fibers?

   How does it look when you pit it against the other fiber implementations in the skynet-benchmark? → https://github.com/atemerev/skynet

Comments are closed.
