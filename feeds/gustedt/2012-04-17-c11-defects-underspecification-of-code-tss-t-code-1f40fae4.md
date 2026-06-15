---
title: 'C11 defects: underspecification of <code>tss_t</code>'
url: https://gustedt.wordpress.com/2012/04/17/c11-defects-underspecification-of-tss_t/
published: "2012-04-17T15:32:04Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1600
---

# C11 defects: underspecification of <code>tss_t</code>

Section 7.26.6 “Thread-specific storage functions” of C11 is severely underspecified since it uses terms that are not introduced (so far) in the context of C. This is really a pity, since POSIX also has `pthread_key_t` that is completely feature equivalent and for which the specification is much more complete.

Jacob Navia had observed that at several occasions in `comp.std.c` but it seems that he had not got enough attention such that this had made it in a defect report.

> The `tss_create` function creates a thread-specific storage pointer with destructor `dtor`, which may be null.

The main problem is that it is nowhere explained/defined what a destructor should be. I think several paragraphs should be added after the one above:

> The effect is that for each thread that has the thread specific storage corresponding to `key` set to a value `x` that is not null, the destructor function `*dtor` is called with `dtor(x)` before the thread exits.

> This call to `dtor` is executed in the context of the same thread; it is sequenced after the `return` statement or the call to `thrd_exit` that terminates the thread and before any return from `thrd_join` of a waiter for this same thread. If there are several thread specific storages for the same thread their destructor functions are called in an unspecific order but with a sequence point between each of these function calls.

> If a destructor function for `key` issues calls to `tss_set`, `tss_get` of `tss_delete` with the same `key` the behavior is undefined.
>
> `tss_set` can be used to set the value of a thread specific storage for a different key `key2` that had not been set before or that has been processed with a call to the corresponding destructor.

By that the set of thread specific storages for a given thread may change during the execution of the corresponding destructors.

> If after processing all tss that are active at the `return` of the thread function or at the end of `thrd_exit` there are still tss that active the procedure of calling destructors is iterated. An implementation may bind the maximum number such of supplementary iterations by `TSS_DTOR_ITERATIONS`.

A second problem is that there are two functionalities that are easily mixed up and which interrelationship should be clarified: the destructor that is called (let us suppose this) at exit of a thread, and `tss_delete` that deletes a thread specific storage for all running threads. I think something like the following should be added in 7.26.6.2 after para 2:

> The deletion of `key` will not change the observable behavior of any of the active threads. If `tss_delete` is called for `key` and there is a thread that has a non-null value for `key` that has passed a terminating `return` statement or call to `thrd_exit` but not yet finished the processing of all its tss destructors, the behavior is undefined.

**Edit:** I now maintain a list of [defect reports and feature requests](http://p99.gforge.inria.fr/defects-and-improvements/ "defect reports and feature requests") together with P99.
