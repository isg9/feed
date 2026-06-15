---
title: The Watermelon Operator
url: https://matklad.github.io/2024/09/24/watermelon-operator.html
published: "2024-09-24T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2024/09/24/watermelon-operator.html
---

# The Watermelon Operator

Sep 24, 2024

In these two most excellent articles, [https://without.boats/blog/let-futures-be-futures](https://without.boats/blog/let-futures-be-futures) and [https://without.boats/blog/futures-unordered](https://without.boats/blog/futures-unordered),

withoutboats introduces the concepts of “multi-task” and “intra-task” concurrency. I want to revisit this distinction — while I agree that there are different classes of patterns of concurrency here, I am not quite satisfied with this specific partitioning of the design space. I will use Rust-like syntax for most of the examples, but I am more interested in the language-agnostic patterns, rather than in Rust’s specific implementation of async.

## [The Two Examples](\#The-Two-Examples)

Let’s introduce the two kinds of concurrency using a somewhat abstract example. We want to handle a `Request` by doing some computation and then persisting the results in the database and in the cache. Notably, writes to the cache and to the database can proceed concurrently. So, something like this:

```
async fn process(
  db: Database,
  cache: Cache,
  request: Request,
) -> Response {
  let response = compute_response(db, cache, request).await;
  spawn(update_db(db, response));
  spawn(update_cache(cache, response));
  response
}

async fn update_db(db: Database, response: Response);
async fn update_cache(cache: Cache, response: Response);

fn spawn(f: impl Future) -> JoinHandle;
```

This is multi-task concurrency style — we fire off two tasks for updating the database and the cache. Here’s the same snippet in intra-task style, where we use join function on futures:

```
async fn process(
  db: Database,
  cache: Cache,
  request: Request,
) -> Response {
  let response = compute_response(db, cache, request).await;
  join(
    update_db(db, response),
    update_cache(cache, response),
  ).await;
  response
}

async fn update_db(db: Database, response: Response) { ... }
async fn update_cache(cache: Cache, response: Response) { ... }

async fn join(
  f: impl Future,
  g: impl Future,
) -> (U, V);
```

In other words:

Multi-task concurrency uses `spawn` — an operation that takes a future and starts a tasks that executes independently of the parent task.

Intra-task concurrency uses `join` — an operation that takes a pair of futures and executes them concurrently as a part of the current task.

But what is the *actual* difference between the two?

## [Parallelism is not](\#Parallelism-is-not)

One candidate is parallelism — with `spawn`, the tasks can run not only concurrently, but actually in parallel, on different CPU cores. `join` restricts them to the same thread that runs the main task. But I think this is not quite right, abstractly, and is more of a product of specific Rust APIs. There *are* executors which spawn on the current thread only. And, while in Rust it’s not really possible to make `join` poll the futures in parallel, I *think* this is just an artifact of Rust existing API design (futures can’t opt-out of synchronous cancellation). In other words, I think it is possible in theory to implement an async runtime which provides all of the following functions at the same time:

```
fn spawn(fut: F) -> JoinHandle
where
  F: Future;

fn pspawn(fut: F) -> PJoinHandle
where
  F: Future + Send + 'static,
  F::Output: Send + 'static;

async fn join(
  fut1: F1,
  fut2: F2,
) -> (F1::Output, F2::Output)
where
  F1: Future,
  F2: Future;

async fn pjoin(
  fut1: F1,
  fut2: F2,
) -> (F1::Output, F2::Output)
where
  F1: Future + Send, // NB: only Send, no 'static
  F1::Output:  Send,
  F2: Future + Send,
  F2::Output:  Send;
```

To confuse matters further, let’s rewrite our example in TypeScript:

```
async function process(
  db: Database,
  cache: Cache,
  request: Request,
): Response {
  const response = await compute_response(db, cache, request);
  const db_update = update_db(db, response);
  const cache_update = update_cache(cache, response);
  await Promise.all([db_update, cache_update]);
  return response
}
```

and using Rust’s rayon library:

```
fn process(
  db: Database,
  cache: Cache,
  request: Request,
) -> Response {
  let response = compute_response(db, cache, request).await;
  rayon::join(
    || update_db(db, response),
    || update_cache(cache, response),
  );
  response
}
```

Are these examples multi-task or intra-task? To me, the TypeScript one feels multi-task — although it is syntactically close to `join().async`, the two update promises are running independently from the parent task. If we forget the call to `Promise.all`, the cache and the database would still get updated (but likely *after* we would have returned the response to the user)! In contrast, rayon feels intra-task — although the closures could get stolen and be run by a different thread, they won’t “escape” dynamic extent of the encompassing `process` call.

## [To await or await to?](\#To-await-or-await-to)

Let’s zoom in onto the JS and the join examples:

```
async function process(
  db: Database,
  cache: Cache,
  request: Request
): Response {
  const response = await compute_response(db, cache, request);

  await Promise.all([
    update_db(db, response),
    update_cache(cache, response),
  ]);

  return response;
}
```

```
async fn process(
  db: Database,
  cache: Cache,
  request: Request,
) -> Response {
  let response = compute_response(db, cache, request).await;

  join(
    update_db(db, response),
    update_cache(cache, response),
  ).await;

  response
}
```

I’ve re-written the JavaScript version to be syntactically isomorphic to the Rust one. The difference is on the semantic level: JavaScript promises are eager, they start executing as soon as a promise is created. In contrast, Rust futures are lazy — they do nothing until polled. And *this* I think is the fundamental difference, it is lazy vs. eager “futures” ( `thread::spawn` is an eager “future” while `rayon::join` a lazy one).

And it seems that lazy semantics is quite a bit more elegant! The beauty of

```
join(
  update_db(db, response),
  update_cache(cache, response),
).await;
```

is that it’s Molière’s prose — this is structured concurrency, but without bundles, nurseries, scopes, and other weird APIs.

It makes runtime semantics nicer even in dynamically typed languages. In JavaScript, forgetting an await is a common, and very hard to spot problem — without await, code still works, but is *sometimes* wrong (if the async operation doesn’t finish quite as fast as usual). Imagine JS with lazy promises — there, forgetting an `await` would *always* consistently break. So, the need to statically lint missing awaits will be less pressing.

Compare this with Erlang’s take on nulls: while in typical dynamically typed languages partial functions can return a value `T` or a `None`, in Erlang the convention is to return either `{ok, T}` or `none`. That is, even if the value is non-null, the call-site is *forced* to unpack it, you can’t write code that happens to work as long as `T` is non-null.

And of course, in Rust, the killer feature of lazy futures is that you can just borrow data from the enclosing scope.

But it seems like there is one difference between multi-task and intra-task concurrency.

## [One, Two, N, and More](\#One-Two-N-and-More)

In the words of withoutboats:

> The first limitation is that it is only possible to achieve a static arity of concurrency with intra-task concurrency. That is, you cannot join (or select, etc) an arbitrary number of futures with intra-task concurrency: the number must be fixed at compile time.

That is, you can do `join(a, b).await`, and

```
join(
  join(a, b)
  c,
).await
```

and, with some macros, even

```
join!(a, b, c, d, e, f).await;
```

but you can’t do `join(xs...).await`.

I think this is incorrect, in a trivial and in an interesting way.

The trivial incorrectness is that there’s `join_all`, that takes a slice of futures and is a direct generalization of `join` to a runtime-variable number of futures.

But `join_all` still can’t express the case where you don’t know the number of futures up-front, where you spawn some work, and only later realize that you need to spawn some more.

This is sort-of possible to express with `FuturesUnordered`, but that’s a yuck API. I mean, even its name screams “DO NOT USE ME!”.

But I do think that this is just an unfortunate API, and that the pattern actually can be expressed in intra-task concurrency style nicely.

Let’s take a closer look at the base case, `join`!

## [Asynchronous Semicolon](\#Asynchronous-Semicolon)

Section title is a bit of a giveaway. The `join` operator *is* `async ;`. The semicolon is an operator of sequential composition: `A; B`

runs `A` first and then `B`.

In contrast, `join` is concurrent composition: `join(A, B)`

runs `A` and `B` concurrently.

And both `join` and `;` share the same problem — they can compose only a finite number of things.

But that’s why we have other operators for sequential composition! If we know how many things we need to run, we can use a counted `for` loop. And `join_all` is an analogue of a counted for loop!

In case where we *don’t* know up-front when to stop, we use a `while`. And this is exactly what we miss — there’s no concurrently-flavored `while` operator.

Importantly, what we are looking for is *not* an async for:

```
async for x in iter {
  process(x).await;
}
```

Here, although there could be some concurrency inside a single loop iteration, the iterations themselves are run sequentially. The second iteration starts only when the first one finished. Pictorially, this looks like a spiral, or a loop if we look from the side:

![](https://github.com/user-attachments/assets/92a9c1dd-1f94-4fac-9e11-5f0c54e6e10e)

What we rather want is to run *many* copies of the body concurrently, something like this:

![](https://github.com/user-attachments/assets/7149b6e8-2d99-4837-911b-36fbee80134a)

A spindle-like shape with many concurrent strands, which looks like wheel’s spokes from the side. Or, if you are *really* short on fitting metaphors:

## [The Watermelon Operator](\#The-Watermelon-Operator-1)

Now, I understand that I’ve already poked fun at the unfortunate `FuturesUnordered` name, but I can’t really find a fitting name for the construct we want here. So I am going to boringly use `concurrently` keyword, which is way too long, but I’ll refer to it as “the watermelon operator” The stripes on the watermelon resemble the independent strands of execution this operator creates:

![wikipedia watermelons](https://github.com/user-attachments/assets/f0760a4a-03ba-45a5-bc26-e9156e28c5c9)

So, if you are writing a TCP server, your accept loop could look like this:

```
concurrently let Some(socket) = listener.accept().await in {
  handle_connection(socket).await;
}.await
```

This runs accept in a loop, and, for each accepted socket, runs `handle_connection` concurrently. There are as many concurrent `handle_connection` calls as there are ready sockets in our listener!

Let’s limit the maximum number of concurrent connections, to provide back pressure:

```
let semaphore = Semaphore::new(16);

concurrently
  let Some((socket, permit)) = try {
    let permit = semaphore.acquire().await;
    let socket = listener.accept().await?;
    (socket, permit)
  }
in {
  handle_connection(socket).await;
  drop(permit);
}.await
```

You get the idea (hopefully):

- In the “head” of our concurrent loop (cooloop?) construct, we first acquire a semaphore permit and then fetch a socket.

- Both the socket and the permit are passed to the body.

- The body releases the permit at the end.

- While the “head” construct runs in a loop concurrently to bodies, it is throttled by the minimum of the available permits and ready connections.

To make this more concrete, let’s spell this out as a library function:

```
async fn join(
  fut1: F1,
  fut2: F2,
) -> (F1::Output, F2::Output)
where
  F1: Future,
  F2: Future;

async fn join_all(futs: Vec) -> Vec
where
  F: Future;

async fn concurrently(condition: C, body: B)
where
  C: FnMut() -> FC,
  FC: FutureOption>,
  B: FnMut(T) -> FB,
  FB: Future;
```

I claim that this is the full set of “primitive” operations needed to express more-or-less everything in intra-task concurrency style.

In particular, we can implement multi-task concurrency this way! To do so, we’ll write a universal watermelon operator, where the `T` which is passed to the body is an `Box>`, and where the body just runs this future:

```
async fn multi_task_concurrency_main(
  spawn: impl Fn(impl Future + 'static),
) {
    ...
}

type AnyFuture = Box<dyn Future + 'static>;

async fn universal_watermelon() {
  let (sender, receiver) = channel::();
  join(
    multi_task_concurrency_main(move |fut| {
      sender.send(Box::new(fut))
    }),
    concurrently(
      || async {
        receiver.recv().await;
      },
      |fut| async {
        fut.await;
      },
    ),
  )
  .await;
}
```

Note that the conversion in the opposite direction is not possible! With intra-task concurrency, we can borrow from the parent stack frame. So it is not a problem to *restrict* that to only allow `'static` futures into the channel. In a sense, in the above example we return the future up the stack, which explains why it can’t borrow locals from our stack frame.

With multi-task concurrency though, we *start* with static futures. To let them borrow any stack data requires unsafe.

Note also that the above set of operators, `join`, `join_all`, `concurrently` is *orthogonal* to parallelism. Alongside those operators, there could exist `pjoin`, `pjoin_all` and `pconcurrently` with the `Send` bounds, such that you could mix and match parallel and single-core concurrency.

## [If a Stack is a Tree, Does it Make Any Difference?](\#If-a-Stack-is-a-Tree-Does-it-Make-Any-Difference)

One possible objection to the above framing of watermelon as a language-level operator is that it seemingly doesn’t pass zero-cost abstraction test. It *can* start an unbounded number of futures, and those futures have to be stored *somewhere*. So we have a language operator which requires dynamic memory allocation, which is a big no-no for any systems programming language.

I think there *is* some truth to it, and not an insignificant amount of it, but I think I can maybe weasel out of it.

Consider recursion. Recursion *also* can allocate arbitrary amount of memory (on the stack), but that is considered fine (I would *also* agree that it is not in fact fine that unbounded recursion is considered fine, but, for the scope of this discussion, I will be a hypocrite and will ignore that opinion of mine).

And here, we have essentially the same situation — we want to allocate arbitrary many (async) stack frames, arranged in a tree. Doing it “on the heap” is easy, but we don’t like the heap here. Luckily, I believe there’s a compilation scheme (hat tip to [@rpjohnst](https://www.abubalay.com) for patiently explaining it to me five times in different words) that implements this more-or-less as efficiently as the normal call stack.

The idea is that we will have *two* stacks — a sync one and an async one. Specifically:

- Every sync function we compile normally, with a single stack.

- Async functions get *two* stack pointers. So, we burn `sp` and one other register (let’s call it `asp`).

- If an async function calls a sync function, the callee’s frame is pushed onto `sp`. Crucially, because sync functions can only call other `sync` functions, the callee doesn’t need to know the value of `asp`.

- If an async function calls another async function, the frame (specifically, the “variables live across await point” part of it) is pushed onto `asp`.

- This async stack is segmented. So, for async function calls, we also do a check for “do we have enough stack?” and, if not, allocate a new segment, linking them via a frame pointer.

- “Allocating a new segment” doesn’t mean that we actually go and call malloc. Rather, there’s a fixed-sized contiguous slab of say, 8 megs, out of which all async frames are allocated.

- If we are out of async-stack, we crash in pretty much the same way as for the boring sync stack overflow.

While this *looks* just like Go-style segmented stacks, I think this scheme is quite a bit more efficient (warning: I in general have a tendency to confidently talk about things I know little about, and this one is the extreme case of that. If some Go compiler engineer disagrees with me, I am probably in the wrong!).

The main difference is that the distinction between sync and async functions is maintained in the type system. There are *no* changes for sync functions at all, so the principle of don’t pay for what you don’t use is observed. This is in contrast to Go — I believe that Go, in general, can’t know whether a particular function can yield (that is, if any function it (indirectly) calls can yield), so it has to conservatively insert stack checks everywhere.

Then, even the async stack frames don’t have to store *everything*, but just the stuff live across await. Everything that happens between two awaits can go to the normal stack.

On top of that, async functions can still do aggressive inlining. So, the async call (and the stack growth check) has to happen only for dynamically dispatched async calls!

Furthermore, the future trait could have some kind of `size_hint` method, which returns the lower and the upper bound on the size of the stack. Fully concrete futures type-erased to `dyn Future` would return the exact amount `(a, Some(a))`. The caller would be *required* to allocate at least `a` bytes of the async stack. The callee uses that contract to elide stack checks. Unknown bound, `(a, None)` would only be returned if type-erased concrete future *itself* calls something dynamically dispatched. So only dynamically dispatched calls would have to do stack grow checks, and that cost seems negligible in comparison to the cost of missing optimizations due to inability to inline.

Altogether, it feels like this adds up to something sufficiently cheap to just call it “async stack allocation”.

I guess that’s all for today? Summarizing:

- Inter-task vs intra-task distinction is mostly orthogonal to the question of parallelism.

- I claim that this is the same distinction as between *eager* and *lazy* futures.

- In particular, there’s no principled obstacles for runtime-bounded intra-task concurrency.

- But we do miss `FuturesUnordered`, but nice. The `concurrently` operator/function feels like a sufficiently low-hanging watermelon here.

- One wrinkle is that watermelon requires dynamic allocation, but it looks like we could just completely upend the compilation strategy we use for futures, implement async segmented stacks which should be pretty fast, and *also* gain nice dynamically dispatched (and recursive) async functions for free?

---

Haha, just kidding! Bonus content! This really should be a separate blog post, but it is tangentially related, so here we go:

## [Applied Duality](\#Applied-Duality)

So far, we’ve focused on `join`, the operator that takes two futures, and “runs” them concurrently, returning both results as a pair. But there’s a second, dual operator:

```
async fn race(
  fut1: F1,
  fut2: F2,
) -> Either
where
  F1: Future,
  F2: Future,
```

Like `join`, `race` runs two futures concurrently. Unlike `join`, it returns only one result — that which came first. This operator is the basis for a more general `select` facility.

Although `race` is dual to `join`, I don’t think it is as fundamental. It is possible to have two dual things, where one of them is in the basis and the other is derived. For example, it is an axiom of the set theory that the union of two sets, `A ∪ B`, is a set. Although the intersection of sets, `A ∩ B` is a dual for union, existence of intersection is *not* an axiom. Rather, the intersection is defined using axiom of specification:

```
A ∩ B := {x ∈ A : x ∈ B}
```

Proposition 131.7.1: `race` can be defined in terms of `join`

The `race` operator is trickier than it seems. Yes, it returns the result of the future that finished first, but what happens with the other one? It gets cancelled. Rust implements this cancellation “for free”, by just dropping the future, but this is restrictive. This is precisely the issue that prevents `pjoin` from working.

I postulate that fully general cancellation is an asynchronous protocol:

1. A *requests* that B is cancelled.

2. B receives this cancellation request and starts winding down.

3. A *waits* until B is cancelled.

That is, cancellation is not “I cancel thou”. Rather it is “I ask you to stop, and then I cooperatively wait until you do so”. This is very abstract, but the following two examples should help make this concrete.

1. A is some generic asynchronous task, which offloads some computation-heavy work to a CPU pool. That work (B) *doesn’t* have checks for cancelled flags. So, if A is cancelled, it can’t really stop B, which means we are violating structured concurrency.

2. A is doing async IO. Specifically, A uses `io_uring` to read data from a socket. A owns a buffer, and passes a pointer to it to the kernel via `io_uring` as the target buffer for a `read` syscall. While A is being cancelled, the kernel writes data to this buffer. If A doesn’t wait until the kernel is done, buffer’s memory might get reused, and the kernel would corrupt some unrelated data.

These examples are somewhat unsatisfactory — A is philosophical (who needs structured concurrency?), while B is esoteric (who uses `io_uring` in 2024?). But the two can be combined into something rather pedestrianly bad:

Like in the case A, an async task submits some work to a CPU pool. But this time the work is very specific — computing a cryptographic checksum of a message owned by A. *Because* this is cryptography, this is going to be some hyper-optimized SIMD loop which definitely won’t have any affordance for checking some sort of a `cancelled` flag. The loop would have to run to completion, or at least to a safe point. And, because the loop checksums data owned by A, we can’t destroy A before the loop exits, otherwise it’ll be reading garbage memory!

And *this* example is the reason why

```
async fn pjoin(
  fut1: F1,
  fut2: F2,
) -> (F1::Output, F2::Output)
where
  F1: Future + Send,
  F1::Output:  Send,
  F2: Future + Send,
  F2::Output:  Send,
```

can’t be a thing in Rust — if `fut1` runs on a thread separate from the `pjon` future, then, if `pjoin` ends up being cancelled, `fut1` would be pointing at garbage. You could have

```
async fn pjoin(
  fut1: F1,
  fut2: F2,
) -> (F1::Output, F2::Output)
where
  F1: Future + Send + 'static,
  F1::Output:  Send + 'static,
  F2: Future + Send + 'static,
  F2::Output:  Send + 'static,
```

but that removes one of the major benefits of intra-task style API — ability to just borrow data.

So the fully general cancellation should be cooperative. Let’s assume that it is driven by some sort of cancellation token API:

```
impl CancellationSource {
  fn request_cancellation(&self) { ... }
  async fn await_cancellation(self) { ...  }

  async fn cancel(self) {
    self.request_cancellation();
    self.await_cancellation().await;
  }

  fn new_token(&self) -> CancellationToken { ... }
}

impl CancellationToken {
  fn is_cancelled(&self) -> bool { ... }
  fn on_cancelled(&self, callback: impl FnOnce()) { ... }
}
```

Note that the question of cancellation being cooperative is *orthogonal* to the question of explicit threading of cancellation tokens! They can be threaded implicitly (cooperative, implicit cancellation is how Python’s trio does this, though they don’t really document the cooperative part (the shields stuff)).

With this, we can write our own `race` — we’ll create a cancellation scope and then *join* modified futures, each of which would cancel the other upon completion:

```
fn race(
  fut1: impl async FnOnce(&CancellationToken) -> U,
  fut2: impl async FnOnce(&CancellationToken) -> V,
) -> Either {
  let source = CancellationSource::new();
  let token = source.new_token();
  let u_or_v = join(
    async {
      let u = fut1(&token).await;
      if token.is_cancelled() {
        return None;
      }
      source.cancel();
      Some(u)
    },
    async {
      let v = fut2(&token).await;
      if token.is_cancelled() {
        return None;
      }
      source.cancel();
      Some(v)
    },
  )
  .await;
  match u_or_v {
    (Some(u), None) => Left(u),
    (None, Some(v)) => Right(v),
    _ => unreachable!(),
  }
}
```

In other words, `race` is but a cooperatively-cancelled `join`!

That’s all for real for today, viva la vida!
