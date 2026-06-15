---
title: 'C11 defects: C threads are not realizable with POSIX&nbsp;threads'
url: https://gustedt.wordpress.com/2012/10/14/c11-defects-c-threads-are-not-realizable-with-posix-threads/
published: "2012-10-14T21:19:37Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1762
---

# C11 defects: C threads are not realizable with POSIX&nbsp;threads

This post is mainly identical to a defect report that hopefully will be discussed by the C standards committee on their next meeting. I found that problem that this raises needs to be better known before people start using this interface more widely, so I decided to also publish it here.

The thread interfaces as they are declared in the `threads.h` header are largely underspecified, such that interpreting them is often just guess-work and leaves room for a wide range of interpretations. This is particularly irritating since there already is an ISO standard about threads that is quite elaborated and mature, namely ISO/IEC 9945:2009, commonly know as POSIX 2010. C11 mentions ISO/IEC 9945:2009, but completely misses to technically relate to it on the thread interface. The semantic specification of C11 threads is in parts so loose, that a stringent implementation of C11 threads on top of POSIX doesn’t seem possible.

Other platforms that are less formalized than POSIX have their own technical restrictions that should additionally be taken into account. The “other platform” for threads that clearly had been targeted by the committee are threads on Microsoft Windows platforms. Most other widely used commodity operating systems are POSIX compatible (from mainframes down to Android phones). But we should not underestimate the potential of the C threads interface. Because it has a reduced interface it might be suitable for a larger range of platforms than we can foresee today. Because C threads don’t enforce a complete share of the address space, such platforms could e.g be accelerators (providing a portable thread interface on GPU?) or networks on chips. The only memory that must be shared by C threads are objects with static storage duration and objects allocated through `malloc` and friends. Thus freestanding environments without `malloc` would only be required to shared statically allocated objects.

In the following I only give an incomplete list of the defects as I noticed them, I suspect that there might be a lot of others.

### List of defects

1. C11-thread functions have an interface that is different from POSIX, namely they return `int` instead of `void*`. Whereas it can be argued that this return type better fits into the overall approach of C, it has the drawback that `pthread_create` can not directly be used for `thrd_create`. In addition to this major drawback, in the specification it is not defined what the term *thread’s result code* would be. Suggestion: the value returned by the thread function or the value passed to `thrd_exit`.

2. Executing a thread function through `thrd_create` is not equivalent to calling that function. Specify that terminating a thread function without a `return` or `thrd_exit` is undefined behavior (exception `main`).

3. Unspecified language for some of the functions. What does “could not be honored” mean compared to “unsuccessful”? This concerns in particular the “signalling” functions. Is a call to `cnd_signal` for which there is no waiter “honoured”?

4. The relation that is established between a `mtx_t` and `cnd_t` is not well specified. Because of that, a C11 conforming implementation would be allowed to perform operations that would be undefined under POSIX. (And thus the C11 functions could not be realized by using POSIX functions.)

   1. May several condition variables concurrently be used with the same mutex? (suggestion: yes)

   2. May a condition variable be used concurrently with different mutexes? (suggestion: no)

   3. The text suggests that a thread returning from a call to `cnd_wait` or `cnd_timedwait` will have regained the lock on the mutex. What happens if “the call could not be honored”? Is there a guarantee that in such case of failure no other thread had been granted in the mean time? What are the permitted cases of failure of these calls?
5. Is there such a thing like “spurious wakeups” on conditions? (Suggestion: yes, otherwise an implementation that would be compatible with POSIX would almost be impossible.)

6. The initialization of `once_flag` doesn’t seem to be mandatory? Can such objects be of automatic storage duration?

7. POSIX has a model of a flat address space that is shared between all threads. In contrast to that, the C standard makes it implementation defined if certain types of objects can be access by different threads than they are defined (thread local, automatic, and probably also temporary lifetime) and it also allows for a segmented address space. The implications of these different models should be worked out more cleanly.

### Suggested Technical Corrigendum

The syntactic specification as it is given in the current version of the standard is suitable, with only some exceptions. (There is also a corresponding report that aims to change the interfaces with no return value.)

The semantic specification is often insufficient, such that it lacks the major goal of this whole thread interface, namely to provide a portable framework for threads. Since a more precise specification here would finally allow certain platforms to provide unambiguously conforming implementations as thin wrappers on top of their native thread models, a corrigendum for these aspects would widen the potential use of the (optional) thread interface.

As a solution to this major problem with the thread interface, I propose to rewrite large parts of the descriptive text, by getting it in sync with ISO/IEC 9945:2009. Being binary compatible with POSIX threads is an imperative for implementations of C11 threads on POSIX platforms.

Since the proposed rewrite would probably need several iterations in the committee, I suggest to preliminary correct this defect by a general statement that relates C threads to POSIX threads.

> ISO/IEC 9945:2009 (POSIX) already defines a multi-threading interface for operating systems with a larger functionality than the threads interface as presented within this standard. The aim of this interface here is to provide portable multi-threading even for platforms that do not conform to POSIX. Concerning functions provided by the interfaces is reduced in comparison to POSIX threads and offers less functionality. On the other hand this standard provides concepts and features that are intended to ease programming of threads and that are necessarily features that are programming language related and that thus have so far not been provided by POSIX. Most important of these is the concept of the sequenced before relation and the specification of atomic operations and data types.

> With one major exception, any incompatibility to ISO/IEC 9945:2009 that would appear by restricting and renaming the POSIX interfaces such that they comply to the C threads interface is unintentional and shall be corrected; any program that would be using the restricted interfaces in conformance to ISO/IEC 9945:2009 and that ensures that the implementation provides the value 1 for the macros `__STDC_THREAD_AUTO_VISIBLE__` and `__STDC_THREAD_TEMPORARY_VISIBLE__` and that makes no implicit assumption about implementation defined behavior that is forced by ISO/IEC 9945:2009 to a certain value shall be a conforming C program. FOOTNOTE:

> A test for such implementation defined behavior could be achieved by

```
#include <threads.h>
#include <stdint.h>
#if !(__STDC_THREAD_AUTO_VISIBLE__ && __STDC_THREAD_TEMPORARY_VISIBLE__ && (CHAR_BIT == 8) && UINTPTR_MAX)
# error "the thread implementation for this platform lacks necessary features"
#endif

```

> On the other hand, implementations of this C standard for threads are allowed to have properties such that they would fail to be complying ISO/IEC 9945:2009 implementations.

> The mentioned major exception is the interface definition given by the function type `thrd_start_t` that is used to describe entry functions to threads. The differing return type for that interface ( `int` instead of `void*` as for POSIX) has been judged by the committee to better suit for the more general C interface.

> The major difference in behavior that is allowed for a C thread implementation that would not be allowed for ISO/IEC 9945:2009 concerns the common address space of threads: objects with automatic storage duration that are defined in one thread are not necessarily visible to any other thread. Whether or not this is the case is implementation defined behavior and must thus be documented by any implementation by means of the feature test macro `__STDC_THREAD_AUTO_VISIBLE__`.

> Other such differences concern the existence of `uintptr_t` (forced by POSIX), the value of `CHAR_BIT` (forced to 8 by POSIX) and the existence of an associated signed type for `size_t`.

Feature test macros that describe the eventual difference to a ISO/IEC 9945:2009 conforming implementation would then be needed. Most of the features that could make up a difference can already be tested through macros, with the exception of the visibility of objects with automatic storage duration. Therefore add to 7.26.1p3:

> `__STDC_THREAD_AUTO_VISIBLE__`
>
> which expands to 1 if objects of automatic storage duration are visible to other threads and to 0 otherwise.

For convenience also add a second macro that makes a similar (non-POSIX feature) observable for conforming programs:

> `__STDC_THREAD_LOCAL_VISIBLE__`
>
> which expands to 1 if objects of thread storage duration are visible to other threads and to 0 otherwise

In accordance with another report also add a feature macro for the implicit POSIX feature:

> `__STDC_THREAD_TEMPORARY_VISIBLE__`
>
> which expands to 1 if objects of temporary lifetime are visible to other threads and to 0 otherwise.

Add a new section to Annex J, portability issues.

> **J.6 C threads and POSIX threads**
>
> The following table lists symbols of the C thread interface that have equivalent interfaces in ISO/IEC 9945:2009 (POSIX). A semantic of the C interfaces that deviates from the corresponding POSIX interface (functions eventually called with analogous or default parameters) is unintentional.

`ONCE_FLAG_INIT``PTHREAD_ONCE_INIT``TSS_DTOR_ITERATIONS``PTHREAD_DESTRUCTOR_ITERATIONS``cnd_t``pthread_cond_t``thrd_t``pthread_t``tss_t``pthread_key_t``mtx_t``phread_mutex_t``once_flag``pthread_once_t``call_once``pthread_call_once``cnd_broadcast``pthread_cond_broadcast``cnd_destroy``pthread_cond_destroy``cnd_init``pthread_cond_init``cnd_signal``pthread_cond_signal``cnd_timedwait``pthread_cond_timedwait``cnd_wait``pthread_cond_wait``mtx_destroy``phread_mutex_destroy``mtx_init``phread_mutex_init``mtx_lock``phread_mutex_lock``mtx_timedlock``phread_mutex_timedlock``mtx_trylock``phread_mutex_trylock``mtx_unlock``phread_mutex_unlock``thrd_create``pthread_create``thrd_current``pthread_self``thrd_detach``pthread_detach``thrd_equal``pthread_equal``thrd_exit``pthread_exit``thrd_join``pthread_join``thrd_sleep``nanosleep``thrd_yield``sched_yield``tss_create``pthread_key_create``tss_delete``pthread_key_delete``tss_get``pthread_getspecific``tss_set``pthread_setspecific`
