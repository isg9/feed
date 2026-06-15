---
title: 'P99 futexes: non-blocking integer valued condition&nbsp;variables'
url: https://gustedt.wordpress.com/2012/11/21/p99-futexes-non-blocking-integer-valued-condition-variables/
published: "2012-11-21T12:18:50Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1822
---

# P99 futexes: non-blocking integer valued condition&nbsp;variables

A while ago I already have written about [Linux futexes](http://gustedt.wordpress.com/2011/01/28/linux-futexes-non-blocking-integer-valued-condition-variables/?preview=true&preview_id=843&preview_nonce=285c5cf50a "Linux futexes") as a really nice concept for a control data structure that goes beyond the ones that we learn or teach in school (mutex, semaphore, condition variable…). I have now gone one step further and integrated futexes into P99; if used on Linux this will evidently use the corresponding Linux feature under the hood, on other platforms a C11 thread implementation using mutexes and condition variables can be used.

One of the real disadvantages of most of the control structures is that they have two very different kinds of events: user events (e.g a call to `cnd_signal`) and system events, often called “spurious wakeups”. Unless we program system code, these spurious wakeups are just an annoyance. They are easily forgotten during development and lead to subtle bugs that only appear on heavy load or when changing the platform and handling them often makes the user code overly complex.

`p99_futex` are designed to work around this type of problems, by still providing a close integration of the control structure into the system and by efficiently distinguishing a “fast path” for operations from a “slow path” where we handle congestion. They provide a counter similar to a conditional variable that allows atomic increments and to wait for it, just as the Linux system call does. (Only that for ideological reasons the base type is an `unsigned`, instead of an `int` as in Linux.)

- a [p99\_futex](http://p99.gforge.inria.fr/p99-html/structp99__futex.html "A counter similar to a conditional variable that allows atomic increment and decrement and to wait fo...") represents an unsigned value

  - the access to the value is atomic

- a [p99\_futex](http://p99.gforge.inria.fr/p99-html/structp99__futex.html "A counter similar to a conditional variable that allows atomic increment and decrement and to wait fo...") represents a lock and wait utility

  - the entrance into the OS wait queue is atomic, a thread only enters the OS wait if the state is what the thread expects
  - it will only wake up threads when the unsigned value has changed, or another thread signals the futex explicitly
  - there are no *spurious wakeups*, state changes of threads only depend on state changes of the value of the futex or user space actions
  - the number of threads that are woken up can be controlled:
    - a minimal number may be used to block (actively!) a thread until enough others are woken up
    - a maximal number may also be given

There are several operations that work on that value similar to their atomic analogues:

- [p99\_futex\_load](http://p99.gforge.inria.fr/p99-html/group__futex_ga94890012858ca4b360a0733de64a0116.html#ga94890012858ca4b360a0733de64a0116 "Obtain the value of futex p00_fut atomically.") returns the value
- [p99\_futex\_fetch\_and\_store](http://p99.gforge.inria.fr/p99-html/group__futex_gae9992e6155e6444297674daf34127bb4.html#gae9992e6155e6444297674daf34127bb4 "Unconditionally and atomically set the futex p00_fut to value p00_desired.") atomically exchanges the value with a new one and returns the old one
- [p99\_futex\_add](http://p99.gforge.inria.fr/p99-html/group__futex_ga805fff4efe80ea7e2553f6abf0fd1621.html#ga805fff4efe80ea7e2553f6abf0fd1621 "increment the counter of p00_fut atomically by p00_hmuch.") atomically adds a quantity to the value
- [P99\_FUTEX\_COMPARE\_EXCHANGE](http://p99.gforge.inria.fr/p99-html/structp99__futex_ad22d35b3a2af941a484a6e06a5e46106.html#ad22d35b3a2af941a484a6e06a5e46106 "a catch all macro to operate on p99_futex") atomically compares the existing value and conditionally exchanges it.

The last three may also be used to wakeup threads that are waiting on the futex, the very last, a macro, may also put the calling thread to a blocking or waiting state.

There is only [p99\_futex\_add](http://p99.gforge.inria.fr/p99-html/group__futex_ga805fff4efe80ea7e2553f6abf0fd1621.html#ga805fff4efe80ea7e2553f6abf0fd1621 "increment the counter of p00_fut atomically by p00_hmuch.") operation and no subtraction. That operation is simply not needed, because adding a negative value in `unsigned` arithmetic will magically do the right thing, here.

If the value of the futex isn’t as expected, a thread may wait ( [p99\_futex\_wait](http://p99.gforge.inria.fr/p99-html/group__futex_ga918f22eb5c3fab10bbc0e27e15da511b.html#ga918f22eb5c3fab10bbc0e27e15da511b "Unconditionally wait for futex p00_fut.")) until another thread notifies it ( [p99\_futex\_wakeup](http://p99.gforge.inria.fr/p99-html/group__futex_ga2f178f4b43246a2860289e04769ada88.html#ga2f178f4b43246a2860289e04769ada88 "Wake up threads that are waiting for a futex.")) about a change of the value. That wait is not (or scarcely) using resources. Such a waiting thread will only be awoken if another thread wants so. Either by calling an explicit [p99\_futex\_wakeup](http://p99.gforge.inria.fr/p99-html/group__futex_ga2f178f4b43246a2860289e04769ada88.html#ga2f178f4b43246a2860289e04769ada88 "Wake up threads that are waiting for a futex."), or implicitly by changing the value through [p99\_futex\_fetch\_and\_store](http://p99.gforge.inria.fr/p99-html/group__futex_gae9992e6155e6444297674daf34127bb4.html#gae9992e6155e6444297674daf34127bb4 "Unconditionally and atomically set the futex p00_fut to value p00_desired."), [p99\_futex\_add](http://p99.gforge.inria.fr/p99-html/group__futex_ga805fff4efe80ea7e2553f6abf0fd1621.html#ga805fff4efe80ea7e2553f6abf0fd1621 "increment the counter of p00_fut atomically by p00_hmuch.") or [P99\_FUTEX\_COMPARE\_EXCHANGE](http://p99.gforge.inria.fr/p99-html/structp99__futex_ad22d35b3a2af941a484a6e06a5e46106.html#ad22d35b3a2af941a484a6e06a5e46106 "a catch all macro to operate on p99_futex").

[P99\_FUTEX\_COMPARE\_EXCHANGE](http://p99.gforge.inria.fr/p99-html/structp99__futex_ad22d35b3a2af941a484a6e06a5e46106.html#ad22d35b3a2af941a484a6e06a5e46106 "a catch all macro to operate on p99_futex") is the most generic tool to operate on a [p99\_futex](http://p99.gforge.inria.fr/p99-html/structp99__futex.html "A counter similar to a conditional variable that allows atomic increment and decrement and to wait fo..."), since it is able to wait until the value is as expected, change it and then notify other threads about the change. It is called with six parameters:

```
#define P99_FUTEX_COMPARE_EXCHANGE(FUTEX, ACT, EXPECTED, DESIRED, WAKEMIN, WAKEMAX)

```

Here, FUTEX is a pointer that is compatible to `p99_futex volatile*` and ACT is an identifier that can be used in the following parameters. If used there it will correspond to local `register` variable of type `unsigned const` holding the actual value of the futex. That variable cannot be modified and its address cannot be taken.

The other parameters are expressions that allow a close integration into the code surrounding the macro call. They may contain arbitrary code that is valid at the point of calling. This much more expressive than what would be possible with an inlined function call: the evaluation may include the local variable *ACT* and thus incorporated the current value of the futex. Here is a pseudo code of what this macro actually does:

```
for (;;) {
  register unsigned const ACT = <load value from FUTEX>;
  if (EXPECTED) {
    if (ACT != (DESIRED)) {
      <try to change value of FUTEX to DESIRED>;
      if (<not successful>) continue;
    }
  } else {
    <block and wait until a change is signaled on FUTEX>
    continue;
  }
  /* Come here only when EXPECTED had been fulfilled
     and the change to DESIRED succeeded */
  unsigned wmin = (WAKEMIN);
  unsigned wmax = (WAKEMAX);
  if (wmin < wmax) wmax = wmin;
  <wakeup at least wmin and at most wmax waiters>
  break;
}

```

Only that each of the macro parameters is expanded exactly once for the C code. As this is a loop structure the code resulting from that argument expansion of *EXPECTED* and *DESIRED* may effectively be evaluated multiple times at run time, in particular when there is congestion on the futex.

### Example 1: a semaphore implementation

To see how to use this, let us look into an implementation of a semaphore, starting with a “post” operation.

- Such a post operation on a semaphore should not place any condition to perform the post action so *EXPECTED* should always be true.

- The operation should increment the value by one, so *DESIRED* should be `ACT + 1`.

- It should always wake up potential waiters, so *WAKEMAX* shouldn’t be `0`.

- It should not wake up more than one waiters to avoid the waiters racing for one little token that had been placed. So *WAKEMAX* should be 1.

```
 inline
 int my_sem_post(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f,
                              // name of the local variable
                              val,
                              // never wait
                              true,
                              // increment the value
                              val + 1u,
                              // wake up at most one waiter
                              0u, 1u);
   return 0;
 }

```

So effectively, such a post operation is just incrementing the value by one and then it up wakes some waiters. In a real world implementation this would better be done by using [p99\_futex\_add](group__futex_ga805fff4efe80ea7e2553f6abf0fd1621.html#ga805fff4efe80ea7e2553f6abf0fd1621 "increment the counter of p00_fut atomically by p00_hmuch.").

A semaphore *wait operation* should block if the value is `0`, and then, once the value is positive, decrement that value by `1` and never wake up any other waiters:

```
 inline
 void my_sem_wait(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f,
                              // name of the local variable
                              val,
                              // block if val is 0 and retry
                              val > 0,
                              // once there is val, decrement it, if possible
                              val - 1u,
                              // never wake up anybody
                              0u, 0u);
   return 0;
 }

```

A semaphore trywait operation should never block, but only decrement the value if it is not `0`, and never wake up any waiters. A trywait operation must be able to return a value that indicates success or failure of the operation. We set the return value by side effect in the *WAKEMIN* expression, where we know that it is evaluated only once.

```
 inline
 int my_sem_trywait(p99_futex volatile* f) {
   int ret;
   P99_FUTEX_COMPARE_EXCHANGE(f,
                              // name of the local variable
                              val,
                              // never wait
                              true,
                              // if there is val decrement by one
                              (val ? 0u : val - 1u),
                              // capture the error by side effect
                              (ret = -!val, 0u),
                              // never wake up anybody
                              0u, 0u);
   if (ret) errno = EAGAIN;
   return ret;
 }

```

### Example 2: reference counting

Another example of the use of an [p99\_futex](structp99__futex.html "A counter similar to a conditional variable that allows atomic increment and decrement and to wait fo...") could be a reference counter. Such a counter can e.g be useful to launch a number of threads, and then wait for all the threads to have finished.

```
 int launch_my_threads_detached(void *arg) {
  p99_futex* fut = arg;
  ...
  my_counter_unlock(fut);
  return 0;
 }

 p99_futex fut;
 p99_futex_init(&fut);
 for (size_t i = 0; i < large_number; ++i) {
   my_counter_lock(&fut);
   thrd_t id;
   thrd_create(&id, launch_my_threads_detached, &fut);
   thrd_detach(id);
 }
 ...
 my_counter_wait(&fut);

```

For that scheme to work we just need three “counter” functions. The first just silently piles up references:

```
 inline
 void my_counter_lock(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f, val, true, val + 1u, 0u, 0u);
 }

```

So effectively, such an operation is just incrementing the value by one. In a real world implementation this would better be done by using [atomic\_fetch\_add](group__atomic_ga9fcd205c9afa613a5ab567b2f0631b87.html#ga9fcd205c9afa613a5ab567b2f0631b87 "Atomically add OPERAND to *OBJP.").

An unlock operation should decrement the value and, if the value falls to `0`, wake up all waiters:

```
 inline
 void my_counter_unlock_unsafe(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f,
                              // name of the local variable
                              val,
                              // never wait
                              true,
                              // decrement by one
                              val - 1u,
                              // no enforced wake up
                              0u,
                              // wake up all waiters iff the value falls to 0
                              ((val == 1u) ? P99_FUTEX_MAX_WAITERS : 0u));
 }

```

This unsafe version makes no provision for underflow of the counter in case these functions are used erroneously. A safer variant would look like this:

```
 inline
 void my_counter_unlock(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f, val,
                              // name of the local variable
                              val,
                              // never wait
                              true,
                              // decrement by one, but only if wouldn't be underflow
                              (val ? val - 1u : 0u),
                              // no enforced wake up
                              0u,
                              // wake up all waiters iff the new value is 0
                              ((val <= 1u) ? P99_FUTEX_MAX_WAITERS : 0u));
 }

```

A wait operation should just expect the counter to fall to `0` and do not much else.

```
 inline
 void my_counter_wait(p99_futex volatile* f) {
   P99_FUTEX_COMPARE_EXCHANGE(f, val, 0u, 0u, 0u);
                              // name of the local variable
                              val,
                              // wait until the value is 0
                              !val,
                              // don't do anything else, no update
                              0u,
                              // and no wake up
                              0u, 0u);
 }

```
