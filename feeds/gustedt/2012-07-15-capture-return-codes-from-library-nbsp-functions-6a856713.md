---
title: Capture return codes from library&nbsp;functions
url: https://gustedt.wordpress.com/2012/07/15/capture-return-codes-from-library-functions/
published: "2012-07-15T21:57:31Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1675
---

# Capture return codes from library&nbsp;functions

C (and POSIX) library functions often communicate error conditions in a two stage procedure: first the return value of the function is set to a special value (such as `0` for `malloc`) and then the caller can read the particular error condition from the pseudo-variable `errno`.

This “traditional” error handling has several disadvantages:

- Code that captures all returns from library functions with conditional code quickly becomes unreadable. Optically such code emphasizes on the error handling part, instead of the normal control flow.
- `errno` is a fragile thing, and in particular after it has been set by a transient error, it must be reset to `0`.

[P99](http://p99.gforge.inria.fr/ "P99") now lets you overload functions with appropriate macros for the error check, and that if an error occurs will either abort execution or throw an error via P99’s [P `99_TROW`](https://gustedt.wordpress.com/2012/02/21/try-throw-and-catch-clauses-with-p99/ "P99_TRY").

To see how that works let us first have a look at an example:

```
#include "p99_try.h"
#define fprintf_throw(...) P99_THROW_CALL_NEG(fprintf, 0, __VA_ARGS__)
#define malloc_throw(...) P99_THROW_CALL_VOIDP(malloc, thrd_nomem, __VA_ARGS__)
#define fclose_throw(...) P99_THROW_CALL_NEG(fclose, 0, __VA_ARGS__)

int main(void) {
  char const* message = "This should be ok.\n";
  int len = fprintf_throw(stdout, "%s", message);
  char* buf = memcpy(malloc_throw(len), message, len);
  fprintf_throw(stdout, "We just printed %d characters, buffer contains: %s", len, buf);
  fclose_throw(stdout);
  fprintf_throw(stdout, "This should abort because the exception is not caught.\n");
}

```

Here we have defined three macros with an extension \_throw derived from library functions. In the case they are successful (up to the `fclose_throw`) they work exactly as the library function. E.g `malloc_throw` returns a pointer to the allocated memory object, just as `malloc`. What differs is the error case, here the last `fprintf_throw` line. The output of the above little program is:

> This should be ok.
>
> We just printed 19 characters, buffer contains: This should be ok.
>
> main:25: fprintf, neg return, exception EBADF=9, library error: Bad file descriptor
>
> Abort (core dumped)

So the return values of the successful function calls are correctly collected in `len` and `buf`. After the closing of `stdout` the last `fprintf_throw` failed. An informative message is printed to the error output and the program is aborted. (The specific code, `EBADF`, may vary according to the OS, here this is on a POSIX system.)

So far we had the program abort execution in case of an error. Often this is an appropriate choice, for functions for which errors are fatal. For library functions that may meet transient errors P99 lets you do more. If we replace the erroneous line by the following try / catch block

```
  P99_TRY {
    fprintf_throw(stdout, "This should be caught.\n");
  } P99_CATCH(int err) {
    if (err) {
      fprintf_throw(stderr, "This is how we catch %s: %s.\n", p99_errno_getname(err), strerror(err));
      p99_jmp_report(err);
    }
  }

```

the output is

> This is how we catch EBADF: Bad file descriptor.
>
> main:26: fprintf, neg return, exception EBADF=9, library error: Bad file descriptor

and execution continues. The last line here is what `p99_jmp_report` produces, namely the error report as it would have been issued if the exception would not have been caught.

There is even another trick, which to my opinion makes the code much more readable. (But I am sure that there will be people who don’t like it.)

```
#include "p99_try.h"
#define fprintf(...) P99_THROW_CALL_NEG(fprintf, 0, __VA_ARGS__)
#define malloc(...) P99_THROW_CALL_VOIDP(malloc, thrd_nomem, __VA_ARGS__)
#define fclose(...) P99_THROW_CALL_NEG(fclose, 0, __VA_ARGS__)

int main(void) {
  char const* message = "This should be ok.\n";
  int len = fprintf(stdout, "%s", message);
  char* buf = memcpy(malloc(len), message, len);
  fprintf(stdout, "We just printed %d characters, buffer contains: %s", len, buf);
  fclose(stdout);
  P99_TRY {
    fprintf(stdout, "This should be caught.\n");
  } P99_CATCH(int err) {
    if (err) {
      fprintf(stderr, "This is how we catch %s: %s.\n", p99_errno_getname(err), strerror(err));
      p99_jmp_report(err);
    }
  }
  fprintf(stdout, "This should abort because the exception is not caught.\n");
}

```

That is, we overload the library function with a macro of the same name. This has several advantages:

- Legacy code doesn’t have to be changed.
- The code is more readable, emphasis is on the “regular” code path.
- Error handling can easily switched on or off by an `#ifdef` conditional compilation.

But what level of error checking you apply in your code is up to your or your company’s coding style:

- explicit error naming conventions with names like `fprintf_throw` or `malloc_or_abort`
- explicit error handling with `P99_TRY` / `P99_CATCH`
- conditional compilation with or without error handling

There are three different error conventions that are captured by specific P99 macros:

- `P99_THROW_CALL_ZERO`, for library functions that return `0` in case of success and some non-zero value in case of failure.
- `P99_THROW_CALL_NEG`, for library functions (such as `printf`) that return `0` or positive values in case of success and negative values in case of failure.
- `P99_THROW_CALL_VOID`, for library functions (such as `malloc`) that return a pointer value. Failure is here a null pointer return or the special value `(void*)-1`.

All three macros receive the name of the library function as a first argument, and a default error condition that should be reported in case that `errno` has not been set to such a value.
