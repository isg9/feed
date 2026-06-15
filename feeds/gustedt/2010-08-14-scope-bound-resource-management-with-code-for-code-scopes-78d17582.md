---
title: Scope Bound Resource Management with <code>for</code> Scopes
url: https://gustedt.wordpress.com/2010/08/14/scope-bound-resource-management-with-for-scopes/
published: "2010-08-14T07:57:08Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=235
---

# Scope Bound Resource Management with <code>for</code> Scopes

Resource management can be tedious in C. *E.g* to protect a critical block from simultaneous execution in a threaded environment you’d have to place a lock / unlock pair before and after that block:

```
pthread_mutex_t guard = PTHREAD_MUTEX_INTIALIZER;

pthread_mutex_lock(&guard);
// critical block comes here
pthread_mutex_unlock(&guard);

```

This is very much error prone since you have to provide such calls every time you have such a block. If the block is longer than some lines it is difficult to keep track of that, since the lock / unlock calls are spread on the same level as the other code.

Within C99 (and equally in C++, BTW) it is possible to extend the language of some sorts such that you may make this easier visible and guarantee that your lock / unlock calls are matching. Below , we will give an example of a macro that will help us to write something like

```
P99_PROTECTED_BLOCK(pthread_mutex_lock(&guard),
    pthread_mutex_unlock(&guard)) {
       // critical block comes here
}

```

If we want to make this even a bit more comfortable for cases that we still need to know the mutex variable we may have something like:

```
GUARDED_BLOCK(guard) {
       // critical block comes here
}

```

The macro `P99_PROTECTED_BLOCK` can be defined as follows:

```
#define P99_PROTECTED_BLOCK(BEFORE, AFTER)                         \
for (int _one1_ = 1;                                               \
     /* be sure to execute BEFORE only at the first evaluation */  \
     (_one1_ ? ((void)(BEFORE), _one1_) : _one1_);                 \
     /* run AFTER exactly once */                                  \
     ((void)(AFTER), _one1_ = 0))                                  \
  /* Ensure that a `break' will still execute AFTER */             \
  for (; _one1_; _one1_ = 0)

```

As you may see, this uses two `for` statements. The first defines an auxiliary variable `_one1_` that is used to control that the dependent code is only executed exactly once. The arguments `BEFORE` and `AFTER` are then placed such that they will be executed before and after the dependent code, respectively.

The second `for` is just there to make sure that `AFTER` is even executed when the dependent code executes a `break` statement. For other preliminary exits such as `continue`, `return` or `exit` there is unfortunately no such cure. When programming the dependent statement we have to be careful about these, but this problem is just the same as it had been in the “plain” C version.

Generally there is no run time performance cost for using such a macro. Any decent compiler will detect that the dependent code is executed exactly once, and thus optimize out all the control that has to do with our variable `_one1_`.

The GUARDED\_BLOCK macro could now be realized as:

```
#define GUARDED_BLOCK(NAME)        \
P99_PROTECTED_BLOCK(               \
    pthread_mutex_lock(&(NAME)),   \
    pthread_mutex_unlock(&(NAME)))

```

Now, to have more specific control about the mutex variable we may use the following:

```
#define P99_GUARDED_BLOCK(T, NAME, INITIAL, BEFORE, AFTER)           \
for (int _one1_ = 1; _one1_; _one1_ = 0)                             \
  for (T NAME = (INITIAL);                                           \
       /* be sure to execute BEFORE only at the first evaluation */  \
       (_one1_ ? ((void)(BEFORE), _one1_) : _one1_);                 \
       /* run AFTER exactly once */                                  \
       ((void)(AFTER), _one1_ = 0))                                  \
    /* Ensure that a `break' will still execute AFTER */             \
    for (; _one1_; _one1_ = 0)

```

This is a bit more complex than the previous one because in addition it declares a local variable `NAME` of type `T` and initializes it.

Unfortunately, the use of `static` for the declaration of a `for`-scope variable is not allowed by the standard. To implement a simple macro for a critical section in programs that would not depend on any argument, [we have to do a bit more than this](https://gustedt.wordpress.com/2011/02/02/handling-control-flow-inside-macros).

Other block macros that can be implemented with such a technique:

- pre- and postconditions
- make sure that some dynamic initialization of a static variable is performed exactly once
- code instrumentation

[P99 now has a lot of examples that use this feature.](http://p99.gforge.inria.fr/p99-html/group__preprocessor__blocks.html)
