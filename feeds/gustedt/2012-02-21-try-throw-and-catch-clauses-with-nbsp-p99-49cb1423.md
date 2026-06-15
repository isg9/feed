---
title: try, throw and catch clauses with&nbsp;P99
url: https://gustedt.wordpress.com/2012/02/21/try-throw-and-catch-clauses-with-p99/
published: "2012-02-21T21:59:11Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1347
---

# try, throw and catch clauses with&nbsp;P99

Before C11 implementing `try/throw/catch` clauses with C alone was difficult. C lacked three features for that. First, there was no clear concept of threads. A `throw` that wants to unwind the stack to a calling function would have to capture the nearest `try` clause on the stack by inspecting a global variable. If you are running in a threaded environment (which most people do theses days) this global variable would have to hold the state of all current `try` clauses of all threads. With the new feature of `_Thread_local` variables (or their emulation through `P99_DECLARE_THREAD_LOCAL` in [P99](http://p99.gforge.inria.fr/)) this is now easy to solve. We just need a thread local variable that implements a stack of states.

The second feature that is useful for such an implementation are atomic operations. Though we can now implement a thread local variable for the stack of states, we still have to be careful that updates of that variable are not interrupted in the middle by signal handlers or alike. This can be handled with atomic operations, P99 also emulates the `_Atomic` feature on common platforms.

The third ingredient is `_Noreturn`. For C11 we can specify that a certain function will never return to the caller. This enables the compiler to emit more efficient code for if a branch in an execution ends with such an `noreturn` function.

These three interfaces together with other features that had been already present in P99, made it straight forward to implement `P99_TRY`, `P99_THROW`, `P99_CATCH` and `P99_FINALLY`.

But let’s just start with a warning. Such a thing in C should not implement the catching of typed exception variables. C has no dynamic type system and implementing such a thing would just blow it up. C error codes are `int`: library functions encode errors through `int` return values, by setting `errno`, also an `int`, or by passing an `int` to `longjmp`. The feature implemented here adds a fourth way to return errors, so again it should stick to the existing and propagate an `int`. In fact, the mechanism that is used under the hood **is** `setjmp/longjmp`, so there is also an implementation imposed reason to use `int` just as everybody else.

The simplest use of this feature is together with `P99_FINALLY`

```
 unsigned char*volatile buffer = 0;
 P99_TRY {
   buffer = malloc(bignumber);
   if (!buffer) P99_THROW(thrd_nomem);
   // do something complicated with buffer
   favorite_func(buffer);
 } P99_FINALLY {
   free(buffer);
 }

```

This will make sure that the buffer allocated in `buffer` will always be freed, regardless what error conditions the code will meet. In particular this will work, even if an exception is thrown from below the call to `favorite_func`. If no exception occurs, the `P99_FINALLY` clause is executed anyhow. Then execution continues after the clause, just as for normal code. If an exception occurs, the clause is executed (and in this case the call to `free` is issued). But afterwards execution will not continue as normal but jump to the next `P99_FINALLY` or `P99_CATCH` block on the call stack.

In the code snippet above the `volatile` qualification of the variable `buffer` is important. `buffer` changes inside the try-block. If we wouldn’t declare it `volatile` we could just see the value of it as we entered the try-block. The jump to the finally-block that is out of the normal control flow might be effective before a store instruction or just use a cached value of the variable. The `volatile` qualifier ensures that such optimizations are inhibited and that we always see the value of `buffer` as it has been stored.

In this example, we could avoid the `volatile` qualification if we don’t change the variable inside the `P99_TRY` block:

```
 unsigned char*const buffer = malloc(bignumber);
 P99_TRY {
   if (!buffer) P99_THROW(thrd_nomem);
   // do something complicated with buffer
   favorite_func(buffer);
 } P99_FINALLY {
   free(buffer);
 }

```

If `buffer is used a lot this would help the optimizer to generate better code.`

`An alternative way to P99_FINALLY is P99_CATCH and to handle different exceptions explicitly. unsigned char*volatile buffer = 0; P99_TRY { buffer = malloc(bignumber); if (!buffer) P99_THROW(thrd_nomem); // do something complicated with buffer } P99_CATCH(int code) { switch(code) { case thrd_nomem: perror("we had an allocation error"); break; case thrd_timedout: perror("we were timed out"); break; } free(buffer); if (code) P99_RETHROW; } The difference here is that we receive the error code through the variable code and we can thus take different action for different exceptional conditions. If it weren't for the P99_RETHROW, the unrolling of the call stack would stop at this point and execution would continue after the catch block. The exception would be considered to be caught. Here, since there is a P99_RETHROW, execution will jump to the next P99_FINALLY or P99_CATCH block on the call stack. In fact a catch clause of P99_CATCH(int code) { // do something here and then if (code) P99_RETHROW; } Would be equivalent to P99_FINALLY { // do something here } only that this wouldn't give access to code. Note that the code depending on P99_TRY must always be an entire block with { } surrounding it. The code depending on P99_FINALLY or P99_CATCH don't has that restriction, it could just be a single statement. The definition of the code variable can be omitted. This can be used to catch any exception and to continue execution after the catch clause in any case: P99_CATCH() { // do some cleanup } // continue here regardless of what happened There is also a "catch all" dialect of all this P99_TRY { // do something complicated that may fail } P99_CATCH(); The ; after the catch is just an empty statement. So this catch clause catches all exceptions, forgets the exception code and does nothing. The raising of an exception is also simple enough. Just use P99_THROW(X) with some error code X. This stops execution at the current point and signals an exception of value X to the next P99_TRY clause that is located on the call stack, if any. If there is no such try clause on the call stack, abort is called. X should be an integer value that by the usual conversions fits into an int. It must be non-zero. Otherwise the arbitrary value 1 as for setjmp/longjmp is transferred. A good convention for the values to throw is using system wide error numbers such as thrd_nomem, ERANGE, EINVAL etc. But any other convention that fits the needs of an application can be used. E.g if you would be using the POSIX regular expression library function regcomp a good idea would be to transfer the error codes that this call gives as return values.`
