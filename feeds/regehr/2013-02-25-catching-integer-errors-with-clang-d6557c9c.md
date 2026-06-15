---
title: Catching Integer Errors with Clang
url: https://blog.regehr.org/archives/905
published: "2013-02-25T03:59:04Z"
feed: regehr
guid: http://blog.regehr.org/?p=905
---

# Catching Integer Errors with Clang

Peng Li and I at Utah, along with our collaborators Will Dietz and Vikram Adve at UIUC, wrote an integer overflow checker for Clang which has [found problems in most C/C++ codes that we have looked at](http://www.cs.utah.edu/~regehr/papers/overflow12.pdf). Do you remember how pervasive memory safety errors were before Valgrind came out? Integer overflows are that way right now.

An exciting recent development is that due to a ton of work done by [Will](http://wdtz.org/), our checker is now in the Clang trunk. Taking a cue from the excellent [address sanitizer](https://code.google.com/p/address-sanitizer/), it is called the integer sanitizer. To use it, [check out and build the latest Clang](http://clang.llvm.org/get_started.html) (you must also build Compiler-RT) and then compile your code using the `-fsanitize=integer` or `-fsanitize=undefined` option. The former will tell you about well-defined but possibly erroneous unsigned overflows, whereas the latter will only tell you about undefined behaviors (including some non-integer-related ones). For more details on these options, see [Clang’s documentation for its code generation options](http://clang.llvm.org/docs/UsersManual.html#controlling-code-generation). The integer sanitizer is not yet in a released version of Clang, but we expect it to be part of the 3.3 release.

One thing we realized very early on in this work is that integer overflows are surprisingly difficult to understand, particularly when they occur in the middle of complex expressions. For example, see this somewhat [undignified interaction between me and the main PHP guy](https://bugs.php.net/bug.php?id=52550). As a result, we put a lot of work into emitting good error messages. Some examples follow.

The integer sanitizer not only checks for divide by zero, which is kind of boring, but also for INT\_MIN / -1 and INT\_MIN % -1. Real codes don’t seem to perform these operations when left alone, but [see here](http://kqueue.org/blog/2012/12/31/idiv-dos/).

```
#include <limits.h>

int main (void) {
  return INT_MIN / -1;
}

```

Result:

**```** **$ clang -fsanitize=integer div.c** **$ ./a.out** **div.c:4:18: runtime error: division of -2147483648 by -1 cannot be represented in type 'int'** **Floating point exception (core dumped)** **$**

**```**

Unsigned overflows are well-defined by C/C++ and are often intentional, particularly in bitsy codes like hash functions and crypto. On the other hand, unintentional unsigned overflows can be bugs, and we can detect them if you want:

```
int main (void) {
  return 0U - 1;
}

```

Result:

**```** **$ clang -fsanitize=integer unsigned.c** **$ ./a.out** **unsigned.c:2:13: runtime error: unsigned integer overflow: 0 - 1 cannot be represented in type 'unsigned int'** **$ clang -fsanitize=undefined unsigned.c** **$ ./a.out** **$**

**```**

Signed integer overflows are undefined by C/C++. Compilers used to provide 2’s complement wraparound for signed overflow, but this is no longer reliable. Therefore, signed overflow should always be avoided. One example commonly seen in real codes is negation of INT\_MIN:

```
#include <limits.h>

int main (void) {
  return -INT_MIN;
}

```

Result:

**```** **$ clang -fsanitize=integer signed.c** **$ ./a.out** **signed.c:4:10: runtime error: negation of -2147483648 cannot be represented in type 'int'; cast to an unsigned type to negate this value to itself** **$**

**```**

Another class of integer error occurs when the right operand to a shift operator is negative or is not less than the bitwidth of the promoted left operand. But these are kind of boring so let’s look at a more arcane kind of shift error: in C99 and later many kinds of signed left shift have undefined behavior, such as this one:

```
int main (void) {
  return 0xffff << 16;
}

```

Result:

**```** **$ clang -fsanitize=integer shift1.c** **$ ./a.out** **shift1.c:2:17: runtime error: left shift of 65535 by 16 places cannot be represented in type 'int'** **$**

**```**

In the more recent versions of C/C++, it is not legal to shift a 1 into, out of, or past the sign bit. See 6.5.7.4 of the [C99 standard](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf) for more details.

Anyway, I think this hits the high points. The slowdown due to integer checking is generally less than 50%. We believe that we can reduce this, but so far have mainly focused on usability and correctness. Will already did some very nice work which marks the trap handling code as cold.

We would appreciate usability feedback from early adopters. On our TODO list are a few things such as:

- Perhaps dropping unsigned overflows from the set of checks enabled by **`-fsanitize=integer`** (these would be enabled by a separate flag).

- Compiler directives for suppressing integer sanitizer errors where they are not wanted.

- Redirecting the error stream to a file or to syslog.

- Porting over a few additional checks from IOC such as detecting lossy truncations and sign conversions.

Let us know if you would find these to be useful.
