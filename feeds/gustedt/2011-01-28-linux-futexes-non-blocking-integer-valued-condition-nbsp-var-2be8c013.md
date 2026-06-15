---
title: 'linux futexes: non-blocking integer valued condition&nbsp;variables'
url: https://gustedt.wordpress.com/2011/01/28/linux-futexes-non-blocking-integer-valued-condition-variables/
published: "2011-01-28T19:57:28Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=843
---

# linux futexes: non-blocking integer valued condition&nbsp;variables

POSIX’ condition variables `pthread_cond_t` unfortunately have some drawbacks:

- They require the use of at least two separate other objects that are only loosely coupled with the condition variable itself:

  - A plain variable, say `X` of type `int`, that actually holds the value of which the condition depends.
  - A mutex that is just to regulate the access to `X` and to the condition variable.
- Generally they are not lock free. To access X we have to

  - lock the mutex

  - inspect X

  - eventually wait for the condition to become true

  - do changes

  - eventually signal other waiters about the changes we made

  - unlock the mutex

Linux’ concept of futexes allows to associate an equivalent concept directly to the variable `X` and to access this variable without taking locks directly in most of the cases.

More precisely, a futex solely works with the *address* and the *value* of variable `X`.

## Atomic access

To access the value, we are supposed to use *atomic* operations. These are operations that are usually implemented in just one assembler instruction (but several clock cycles). They access the variable, change it and return the previous value with guaranteed atomicity: no other thread or process will come in the way and if the operation succeeds, all others will see the changed value. I will not go much into detail of these operations. They are not part of the current C99 standard but will be in the upcoming C1x. I will use the notations from there, but you may have a read into the [gcc extension](http://gcc.gnu.org/onlinedocs/gcc/Atomic-Builtins.html#Atomic-Builtins) that already implements these kind of operations. I will only suppose that we are given two functions

```
inline int atomic_fetch_add(int volatile *object, int operand);
inline int atomic_fetch_sub(int volatile *object, int operand);

```

Both return the value that had previously been stored in `*object` and add (respectively subtract) the operand.

## Making futexes available to user space

As said above, Linux’ futexes work with the address and the value of an `int` variable `X`. Linux exports several system calls for that; we will look into two of them that will offer a “ *wait*” and a “ *wake*” procedure.

If you will look into the man page for futex on a Linux machine you will usually find and entry ( *futex(2)*) but unfortunately the library interface for that system call is missing so we have to build it first. The man page also states that you should stay away from this feature if you don’t know what you are doing. The goal of this text here is actually to make sure that at the end you will know what is necessary to use futexes well.

For what we will be doing here, we will encapsulate the system call as follows

```
# include <linux/futex.h>
# include <sys/syscall.h>
inline
int orwl_futex(int *uaddr, /*!< the base address to be used */
               int op,     /*!< the operation that is to be performed */
               int val     /*!< the value that a wait operation
                             expects when going into wait, or the
                             number of tasks to wake up */
               ) {
  return syscall(SYS_futex, uaddr, op, val, (void*)0, (int*)0, 0);
}

```

## Wake up

A simple wakeup function now looks like this

```
inline
int orwl_futex_wake(int* uaddr, int wakeup) {
  int ret = orwl_futex(uaddr, FUTEX_WAKE, wakeup);
  return ret;
}

```

The parameter `uaddr` is the address to which the futex is associated. Internally the kernel has a hash table with all (kernel space) addresses for which there are waiters. Calling `orwl_futex_wake` will now wake up `wakeup` threads that the kernel has registered as waiting for that address, respectively less if there are not as much waiting, or just nobody if nobody is waiting.

As indicated, the kernel is using its internal memory address for the identification and not the virtual address that it receives through the system call. Thereby the futex mechanism works not only between different threads of the same process, but also between any processes that have mapped the same kernel page into their virtual memory.

Two commonly tasks for wakeups are these:

```
inline
int orwl_futex_signal(int* uaddr) {
  return orwl_futex_wake(uaddr, 1);
}
inline
int orwl_futex_broadcast(int* uaddr) {
  return orwl_futex_wake(uaddr, INT_MAX);
}

```

That’s it for the wake up part. Since waking up others cannot produce deadlocks this part is really simple.

## Waiting

Waiting is a bit more complicated, because here we have to offer tools that can give us guarantees that we will not have deadlocks:

```
inline
int orwl_futex_wait_once(int* uaddr, int val) {
  int ret = 0;
  if (P99_UNLIKELY(orwl_futex(uaddr, FUTEX_WAIT, val) < 0)) {
    int errn = errno;
    if (P99_UNLIKELY(errn != EWOULDBLOCK && errn != EINTR)) {
      ret = errn;
    }
    errno = 0;
  }
  return ret;
}

```

As you can see the wrapper around `orwl_futex` is a bit more complex, because the idea that we have of the value `(*uaddr)` may be wrong. A scenario that could mess up our wait procedure:

- We lookup `(*uaddr)` and find that it has value `5`
- We conclude that `5` for us is really a bad value and decide to take a nap.
- Before we are able to call a “wait” function we are de-scheduled by scheduler because our time slice is over.
- Another process or thread changes `(*uaddr)` to `6` and wakes up all waiters.
- We are scheduled again and make the “wait” call, just to never wake up again.

`orwl_futex_wait_once` will return under three different circumstances:

- A wake up event is triggered by some other task or process. This is in fact what we are waiting for.
- The value `(*uaddr)` is initially not equal to `val`. Somebody has change the value in the mean time and the condition that made want to wait is no longer fulfilled.
- An interrupt is received. The system wakes up the thread to handle a signal that it received. Once the interrupt handler returns it returns to its current context, that is to the end of the wait.

So once we come back from such a call we can not be sure that the world as we know it has improved, we always have to check. Typically `orwl_futex_wait_once` is therefore wrapped into some loop that checks the condition that interests us and returns to the wait state if it is not yet satisfactory. Because we want to be able to handle general expressions for our condition, we wrap that loop in a macro and not an `inline` function:

```
#define ORWL_FUTEX_WAIT(ADDR, NAME, EXPECTED)                   \
do {                                                            \
  register int volatile*const p = (int volatile*)(ADDR);        \
  for (;;) {                                                    \
    register int NAME = *p;                                     \
    if (EXPECTED) break;                                        \
    register int ret = orwl_futex_wait_once((int*)p, NAME);     \
    if (P99_UNLIKELY(ret)) {                                    \
      assert(!ret);                                             \
    }                                                           \
  }                                                             \
 } while (false)

```

But as you hopefully see the inner loop is still relatively simple. It loads `(*uaddr)` into a local variable NAME, checks the condition EXPECTED and only goes into the wait function if it is not fulfilled. When coming back from the wait, the procedure is iterated in the hope that we now fulfill the condition.

The rest of the macro is some sort of syntactic sugar that ensures that ADDR as passed to the macro is only evaluated once and also, that this macro can be placed anywhere a simple C statement would do.

## A counter example

Let us illustrate all this with a simple example of a reference count. The goal is a data structure that has

- atomic increment and decrement for those threads that refer to a resource and
- a wait function for a maintenance thread that waits until all referrers have released the resource.

An API could look like this

```
typedef struct counter counter;
struct counter { int volatile val; };
inline int counter_inc(counter* c);
inline int counter_dec(counter* c);
inline void counter_wait(counter* c);
#define ACCOUNT(COUNT) P99_PROTECTED_BLOCK(counter_inc(&COUNT), counter_dec(&COUNT))

```

The first two functions are just simple wrappers around the atomic operators that are given above, you’ll easily figure that out yourself. We’d just have to come up with a sensible specification if we want to return the value of the counter before or after the operation. We don’t even have to use them directly in user code but through the macro ACCOUNT we may use them as

```
counter myCount = { 0 };

ACCOUNT(myCount) {
   // do something

}

```

Here [P99\_PROTECTED\_BLOCK](http://p99.gforge.inria.fr/p99-html/group__preprocessor__blocks.html) will do all the wrapping to make sure that the “inc” and “dec” functions are called exactly once before entering and after leaving the inner block.

Now last but not least, the implementation of the wait function is just wrapping around our macro:

```
inline void counter_wait(counter* c) {
  ORWL_FUTEX_WAIT(&(c->val), X, !X);
}

```

Here the role of `X` is just to give a name to the internal variable such that we may refer to it inside the expression. The expression here is just `!X` to test if `X` is `0` or not. We could have placed any other expression such as `(X <= 0)` or `(X % 42)` of our liking, there. Anything that lets us interpret the value of the counter as a condition that we want to see to be fulfilled.
