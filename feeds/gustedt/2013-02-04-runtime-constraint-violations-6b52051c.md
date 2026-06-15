---
title: Runtime-constraint violations
url: https://gustedt.wordpress.com/2013/02/04/runtime-constraint-violations/
published: "2013-02-04T08:30:23Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1893
---

# Runtime-constraint violations

Somewhat hidden in Annex K, C11 introduces a new term into the C standard, namely *runtime-constraint violations*. They offer an important change of concept for the functions that are defined in that annex: if such a function is e.g called with invalid parameters, a specific function (called *runtime-constraint handler*) is called, that could e.g abort the program, or just issue an error message. This is in sharp contrast to the runtime error handling in the rest of the C standard, where the behavior under such errors is mostly *undefined* (anything may happen then) or sometimes reported to implementation defined behavior (and thus poorly portable and predictable).

Annex K, obscurely coined “ *Bounds checking interfaces*“, introduces some `typedef` and a series of replacement functions for many C library functions. The function names in this series are usually derived from the name of the function they replace and by adding the suffix `_s` to the function name, e.g the function `qsort` gets a “secure” twin interface called `qsort_s`, as we have seen [in an earlier post](https://gustedt.wordpress.com/2012/12/04/inline-functions-as-good-as-templates/ "inline functions as good as templates").

As a first example let us look into the relevant part of the specification of `qsort_s`:

> **Runtime-constraints**
>
> Neither `nmemb` nor `size` shall be greater than `RSIZE_MAX`. If `nmemb` is not equal to zero, then neither `base` nor `compar` shall be a null pointer.
>
> If there is a runtime-constraint violation, the `qsort_s` function does not sort the array.

All the constraint violations that are mentioned here, would lead to *undefined behavior* if they occurred with `qsort` and not with `qsort_s`. For the “ `_s`” functions the programmer has a choice of what will happen if any of these events occurs. Two runtime-constraint handlers are always guaranteed to exist, `abort_handler_s` and `ignore_handler_s`. They perform the action that you would expect from their names: the first aborts the program whenever a runtime-constraint occurs and the second just ignores all such events.

## Runtime-constraint handlers

A program may declare its own runtime-constraint handlers at any time, simply through a call to `set_constraint_handler_s`. They must just follow the interface specification that is given by the `constraint_handler_t` function pointer type:

```
typedef void (*constraint_handler_t)(const char * restrict msg, void * restrict ptr, errno_t error);

```

That is the constraint handler receives an informative message string, a pointer to an object that may contain information about the context in which the violation occurred (or that can be 0), and the error code that the function would or will return. A simple handler could look as follows:

```
void simple_constraint_handler(const char * restrict msg, void * restrict ptr, errno_t error) {
  if (stderr) {
    if (ptr) fprintf(stderr, "A runtime-constraint of type %d occurred with object %p\n", error, ptr);
    else fprintf(stderr, "A runtime-constraint of type %d occurred\n", error);
}

```

A more sophisticated one could like this:

```
void informative_constraint_handler(const char * restrict msg, void * restrict ptr, errno_t error) {
  if (stderr) {
    size_t len = strerrorlen_s(error);                            // has no constraint violations
    char estr[len + 1];
    constraint_handler_t previous = set_constraint_handler_s(0);  // restore the default handler
    strerror_s(estr, len, error);
    if (ptr) fprintf(stderr, "A runtime-constraint of type %d occurred with object %p: %s\n", error, ptr, estr);
    else fprintf(stderr, "A runtime-constraint of type %d occurred: %s\n", error, estr);
    set_constraint_handler_s(previous);                           // restore the previous handler (that should be this one here)
  }
}

```

But much more would be possible, e.g to abort on certain error codes and ignore others.

Since an runtime-constraint handler may be just set up to ignore the constraint violation, all functions that return an `errno_t` should still be checked if the call was on error, and the Annex K functions give still some guarantees on the state of the application after the violation. As we have seen above `qsort_s` guarantees that the array to be sorted has not been touched under the presence of a runtime-constraint violation, and it returns an appropriate error code to let the application program know what went wrong.

## Two new typedefs

This annex also defines two new typedefs. They are `errno_t` which is alias for `int` and `rsize_t` for `size_t`. Both are meant to mark a semantic difference from the type they alias; `errno_t` should only be used for error returns and all library functions with that return type are guaranteed to return a valid error number.

`rsize_t` is supposed to only hold values that correspond to possible sizes of objects on the platform and there is a macro `RSIZE_MAX` for the maximum value of it. For example there is currently no machine with 64 bit wide `size_t` that could hold an object that would be as large as `SIZE_MAX`. So a function seeing a value that is close to `SIZE_MAX` that is supposed to represent the size of an object can be pretty sure that the value is erroneous and probably the result of a wrap around of some negative value. With this reasoning, on modern machines `RSIZE_MAX` can be set to `(SIZE_MAX/2)` and with that “wrap arround” errors on size quantities can easily be detected.

The functions of Annex K that receive arrays as parameters also receive their size as an `rsize_t` and will check for this type of out-of-bounds error. (Probably the annex got its strange title from this idea.) Also most of them will check if pointer parameters are null pointers.

## Other interfaces

Beyond parameter bounds checking, the Annex K interface provides more features.

- We already have seen `qsort_s` that not only does this type of bounds checking but also adds a *context* pointer to the comparison interface, such that it becomes easier to use as general purpose sorting algorithm. In particular, this approach eliminates the need to communication to comparison functions through global variables. Similar rules hold for `bsearch_s` that augments the `bsearch` function as previously known.
- Augmented interfaces from <stdio.h> forbid dereferencing of null pointers or writing with the “%n” format, such that buffer overflow attacks become more difficult.
- There are string handling functions that complete the interfaces from <string.h>. E.g `memcpy_s` and `strncpy_s` additionally check if the two input arrays or strings overlap; `strerrorlen_s` and `strlen_s` offer overrun safe functions to check the length of strings.
- functions for date and time
- multibyte and wide character utilities

## Implementation

There seems some ideological thinking around that Annex K, which is not related to its contents but to its originator. This really is a pity, because Annex K is relatively straight forward in its ideas and interfaces. And really it isn’t too difficult to put into code. [P99](http://p99.gforge.inria.fr/ "P99") already implements a lot of it with simple wrappers around other library functions. The main thing that it doesn’t do (and probably never will) is the restriction on the possible `printf` (and similar) formats. They would need additions to the sources of these functions and can not be implemented with macros or alike.

P99 just wraps the functions that it amends by macro wrappers. By that it is able to give even more diagnostics of a runtime-constraint violation than is required by Annex K. Often it is even able to trace the function and its calling context in the diagnostic output that the constraint handlers provide.

Other than the wrappers in P99, I have only heard of one implementation of these functions in the context of publicly available open source C libraries, but I’d happily list more, just drop me a line:

- [SLIBC](http://code.google.com/p/slibc/ "SLIBC") provided by SBA Research, Austria.
