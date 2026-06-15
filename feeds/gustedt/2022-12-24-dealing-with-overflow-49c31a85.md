---
title: Dealing with overflow
url: https://gustedt.wordpress.com/2022/12/24/dealing-with-overflow/
published: "2022-12-24T15:41:18Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3887
---

# Dealing with overflow

In the [previous post](https://gustedt.wordpress.com/2022/12/18/checked-integer-arithmetic-in-the-prospect-of-c23/) we have seen that C23 will provide us with tools for efficient overflow check for arithmetic. When discussing this, several people have asked why these tools do not provide a specific model to *handle* overflow, such as aborting or raising a signal. The answer to this is actually quite simple:

**There is no commonly agreed strategy to handle overflow.**

So for the C committee (and for the gcc feature this is based upon) there was no obvious choice for a strategy. Instead, the features provide the basics to implement any such strategy based on *policy decisions* that will be dependent on application specific settings, be they technical or cultural

**Unadorned overflow in C**

Let’s start with the typical example that shows different strategies and expectations that are usually associated with arithmetic overflow in C:

```
void doSomething(T);

for (T i = 1; i; ++i) {
  doSomething(i);
}

```

The first observation is that the status of this code depends on

- The type `T`, whether it is a signed type or an unsigned type.
- If `doSomething` has a side effect.
- If `doSomething` will terminate the execution for some positive value of `i`.

The possible behavior of this code is the following.

- If `T` is an unsigned type and `doSomething` has a side effect, the loop is always well-defined. It continues execution, as long as no call to `doSomething` terminates execution. If it finally reaches the maximum value, the `++` operation wraps around, `i` is `0` and the loop terminates.
- If `T` is a signed type and `doSomething` has a side effect, the loop continues until a call terminates execution. If no positive value `i` has that effect and the maximal value for `i` is reached, the behavior is undefined.

**Absence of side effects**

If we do not specify more then the prototype, the compiler cannot know if it has side effects, and so the executable has to behave as described. But if `doSomething` has no side effect, in C23 it can be annotated with an attribute:

```
[[unsequenced]] void doSomething(T);

```

Then, we’d know that for an unsigned type `T` the loop is finite with no side effects, and so the compiler could remove it entirely.

In contrast to that, if `T` is signed, we’d know that after a finite number of steps the maximum value would be reached and the behavior is not defined. So any execution that leads to the loop is undefined, and the loop is (in C23) as if it had been written

```
unreachable();

```

That is, it is supposed that the whole branch of execution in which the loop is found will never be reached.

**Choosing a policy**

The point I want to make here is that because of the innocent looking `++` operation, the code can have quite different behavior and the execution may have very different outcome. Which of these behaviors or outcome is intended? We simply don’t know. Generally supposing such an intent is as reliable as reading tea leaves.

Now consider a function that guarantees a specific behavior when overflow occurs

```
T add_overflow_T(T a, T b) {
    T ret;
    if (ckd_add(&ret, a, b)) {
        OVERFLOW_HANDLER();
    }
    return ret;
}

```

and rewrite our loop to

```
for (T i = 1; i; i = add_overflow_T(i, 1)) {
  doSomething(i);
}

```

Here, a macro `OVERFLOW_HANDLER()` represents a policy that we apply to overflow. It can be different for different applications or contexts and can be chosen for each translation unit separately. Here are some possible choices for the macro:

- `(void)0` ignores overflow. The result of the operation is just the wrapped value. For an unsigned type `T` it is `0` and for a a signed one it is the minimum value. This choice has the same semantics as the unsigned case, above. The behavior is always defined and execution of the loop stops when the hole value range of `T` has been explored.
- `unreachable()` makes the behavior on overflow undefined. This choice has the same semantics as the signed case, above. Thus this is only defined if `doSomething` is know to terminate for some argument. This policy should only be chosen if the application code can guarantee that.
- `abort()` crashes the execution when overflow occurs. This is quite different from the previous one, because this always has defined behavior. It should only be used in situations where crashing the application is tolerable, in particular where none of the repair work of the system is needed. *This is almost always the wrong choice.*
- `exit(EXIT_FAILURE)` is a much nicer failure mode on overflow than `abort` and guarantees that application specific exit handlers (registered with `atexit`) are called and system cleanup is performed.
- `quick_exit(EXIT_FAILURE)` is a variant of the above that also calls application specific handlers ( `at_quick_exit`) and performs some quick system cleanup.
- `raise(SIGFPE)` raises a signal when overflow occurs. Such signal can be caught by a signal handler, and some repair action could be performed there, that could, in principle, lead to a continuation of the execution.

None of these policies is “good” or “bad” and any of them has a valid field of application. Consequently the new C standard doesn’t select one and leaves the choice and responsibility to the application.
