---
title: Simple C11 atomics:&nbsp;atomic_flag
url: https://gustedt.wordpress.com/2012/01/22/simple-c11-atomics-atomic_flag/
published: "2012-01-22T13:37:35Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1281
---

# Simple C11 atomics:&nbsp;atomic_flag

C11 (and similar C++11) has a new primitive type in its new toolbox for atomics: `atomic_flag`. As you’d perhaps imagine from the name, a variable of that type has two different states, called “cleared” and “set”, and access to it is atomic. It is even guaranteed to be “lock-free”, see below.

Sounds like [`atomic_flag`](http://p99.gforge.inria.fr/p99-source/p99-html/structatomic__flag.html) is the same as [`_Atomic(_Bool)`](http://gustedt.wordpress.com/2012/01/17/emulating-c11-compiler-features-with-gcc-_atomic/), doesn’t it? Not quite. Access to it is even more restricted than for a `_Bool`, however it might be more efficient.

The wish list for atomic operations are

- They should be implemented efficiently. Generally one special assembler instruction should suffice
- They should be lock-free. The C11 standard doesn’t explain much what that is. The only effect that is described that lock-free operations have on the abstract state machine is that lock-free operations are perceived by the machine as stateless. Interactions between different parts of the program and the environment (threads, signal handlers) are coherent. An atomic store, e.g., has taken place or not, and the knowledge about that can be deduced from the state of the machine.

These two requirements do not always work well together. On some processors atomic primitives are not stateless. They are composed operations, where

1. first some sort of lock on the variable is placed,
2. the required operation is performed,
3. the lock is released.

if the thread is interrupted or unscheduled somewhere between 1. and 3. the thread doesn’t know if the operation has been performed or not. Another thread that wants to get access to the resource is blocked because the lock is already taken.

An `atomic_flag`, besides non-atomic initialization, only provides two simple operations, `atomic_flag_test_and_set` and `atomic_clear`, that can be more efficiently implemented in processors. In particular one important operation is missing that we usually assume on any data type: a load operation or if you want simple evaluation in an expression.

> The state of an atomic\_flag cannot be determined without affecting it.

- `atomic_flag_test_and_set` returns the previous state of the flag *and* sets it at the same time.
- `atomic_clear` clears the flag unconditionally.

A typical use of an `atomic_flag` to protect an `unsigned` for an atomic operation would look like

```
static atomic_flag cat = ATOMIC_FLAG_INIT;
static unsigned volatile counter= 0;
.
do {} while (atomic_flag_test_and_set(&cat));
++counter;
atomic_flag_clear(&cat);

```

Here the `do/while` loop actively spins round until it is detected that `cat` had previously been “clear”. (Now we *know* that it is set, since it was this thread who set it.)

Once we know that it had been clear before, we also know that we are the only one to know and can perform the operation on the protected variable, here an increment. Then we clear `cat` such that other threads (or the same thread but from a signal handler) can get access to the counter in turn.

> C11 requires that the operations on `atomic_flag` guarantee the necessary consistency for all memory operations.

In particular, here it guarantees that the operation on `counter` “sees” all previous stores to that variable and that all access to it that comes later will see the incremented value.

The type of lock we are using here is called a spinlock. A spinlock is from the semantic view of the program not very different from a mutex. The difference is the implementation: it doesn’t need OS support (or almost, see below). It is very fast in the non-contended case, the so-called fast-path. When there is nobody else fighting for the lock it is just one instruction resulting in a bus-transfer. Nothing else.

Most, or at least many, programs with locks don’t need much more than that. Having multiple threads fighting for the same resource is rare.

But if there are multiple threads going for the same resource, what do we lose? Not much, as long as the protected critical section is small. In our example above this would be just the load, increment and store of `counter`. The chance that a signal or unscheduling will just take place during that short time is very small. Such a thing might happen from time to time, and if it happens at least one whole scheduling time slice will be wasted. So on average the atomic add above will be very fast, with some outliers here and there.

Another use of `atomic_flag` is the management of a resource that is shared between two threads:

```
typedef struct prot_string prot_string;
struct prot_string {
  atomic_flag cat;
  char volatile* string;
};

```

A function to release such a beast could look as follows. The `str` is only deallocated if the other thread has called the same function already, since then the state of `cat` is already set. Otherwise, the only side effect of `free_prot_string` is the setting of the flag. But nobody else than the calling thread is to know about that.

```
prot_string* prot_string_init(prot_string* str) {
  if (str) {
    atomic_flag_init(&str->cat);
    str->string = calloc(20, 1);
  }
  return str;
}
void prot_string_free(prot_string* str) {
  if (atomic_flag_test_and_set(&str->cat)) {
    free(str->string);
    free(str);
  }
}

```

A worker thread, say, could now receive such `prot_string` as a parameter from his caller.

```
int work_thread(void *arg) {
  prot_str* str = arg;
  /* ... do the work and go away ... */
  prot_string_free(str);
  return 0;
}

```

```
int main(void) {
  prot_string * str = malloc(sizeof *str);
  prot_string_init(str);

  thrd_create(..., work_thread, str);

  /* ... detach the thread ... */

  free_prot_string(str);
}

```

Both the worker thread and the `main` function would try to release the protected resource at the end. For the programmer it could be hard to predict which of the two would need to get access to the resource (here our variable `str`). If the worker thread happens to be fast, it might be the first to come to the end to destroy `str`. Or it might be hold back by some signal, expensive calculation or whatever.

Only the last one would then really free the two `malloc` ed objects safely. The use of the `atomic_flag` allows to decide clearly which one is responsible at the end for destroying the shared resource: the one that receives a `0` is the first to leave and mustn’t destroy, where the one that receives a 1 knows that the other has already left and that `str` can be safely destroyed.
