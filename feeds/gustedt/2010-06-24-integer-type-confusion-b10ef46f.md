---
title: Integer type confusion
url: https://gustedt.wordpress.com/2010/06/24/integer-type-confusion/
published: "2010-06-24T12:59:05Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=104
---

# Integer type confusion

Many people still use `int` as *the* integer type to be used in C. Don’t do that. This is misleading, non portable (yes!) and error prone.

- Valid use of `int` is for the return code of a system function or `errno`.
- Valid use of `char` is for the character in a string.
- Valid use of `char*` is *byte arithmetic* on pointers.

That’s it really.

Traditionally, integer types in C are a mess, already syntactically. You can have combinations such as `long long unsigned int` etc that make the code really hard to read. For the following we will use the 10 different real primitive types that C99 provides and for 6 of them use `typedef` to have an abbreviation for each of them

```
typedef signed char schar;
typedef unsigned char uchar;
typedef unsigned short ushort;
typedef unsigned long ulong;
typedef signed long long sllong;
typedef unsigned long long ullong;

```

To a `signed int` we will just refer as `signed` and to an `unsigned int` as `unsigned` and we will use `short` and `long` for the other two. All these 10 refer to different types for the compiler but may not necessarily be of a distinct width, or of just the width you suppose them to be. No, `int` is not always 32 bit wide, no, `long` is not always 64 bit wide, be carefull.

There is another integer type in C99 that doesn’t get too much attention, `_Bool` (or `bool` if you include *“stdbool.h”*). This blog entry here doesn’t go much into the details of Boolean logic, but you should use `bool` whenever it is appropriate for a variable that has the semantics of a *truth value*.

## Pitfall Number One: array indexing

Arrays in C are indexed from *0* up to *n-1*, where *n* is the number of elements in the array, you all know that. So indexing an array with a negative number is something odd, that should only happen under very controlled circumstances.

**Never use a signed value to index an array or to store its bounds**

unless you are certain of what you are doing. But just as a reflex, don’t use `int`, `signed` or something similar.

Use the unsigned integer type of your system that has the correct width to address any size of object that is possible on your system. This type has a name: `size_t`. Use it, it is as simple as that.

**Use `size_t` to index an array or to store its bounds**

By that your code becomes portable since this is well defined on every modern system and you are much less vulnerable to overflow problems.

## Pitfall Number Two: type width

First, the *width* of a type (even if it is unsigned) is not forcibly in direct relation with its *size* as returned by the `sizeof` operator: there may unused bits, so-called padding bits.

Then, don’t make false assumptions on the width of any of the 10 integer types. Such assumptions are the main source for 32 versus 64 bit portability problems. The only reasonable assumptions that you can make are for the size

```
1 = sizeof(schar) <= sizeof(short) <= sizeof(signed) <= sizeof(long) <= sizeof(sllong)
1 = sizeof(uchar) <= sizeof(ushort) <= sizeof(unsigned) <= sizeof(ulong) <= sizeof(ullong)

```

There are some other properties about the minimum width of these types, but if you stick to the following you wouldn’t need them.

Whenever you use an integer as bitfield think twice of what you assume about the number of bits of the underlying type. Same holds, obviously, if you assume something about the maximum or minimum value of the type. The `[u]intX_t` family of `typedef` that is guaranteed to exist for C99 is there for you. Here the *X* stands for the width that you want, typical values for *X* are *8*, *16*, *32*, and *64*. E.g `uint64_t` is an unsigned integer type of exactly *64* bits. And so you see also there are 4 such commonly used width but 5 different types. So usually *there are* at least two basic types with the same width.

**Use `[u]intX_t` whenever you make an assumption about the width of a type**

**Edit:** Well, the `[u]intX_t` types are only required by POSIX, C99 leaves them optional. C99 only requires the types `[u]int_fastX_t` to exist. On most commodity architectures `[u]intX_t` will exist nowadays, but if you want to be sure that your code also is valid on embedded platforms and alike, use `[u]int_fastX_t`.

If you need the maximum value of an unsigned integer type `T`, `(T)-1` will always do the trick, no need to know about magic constants. If you need to know if a type is signed or unsigned, `((T)-1 < (T)0)` will do. Since these are a bit ugly, I use the following macros

```
#define P99_0(T) ((T)0)
#define P99_M1(T) ((T)-1)
#define P99_ISSIGNED(T) (P99_M1(T) < P99_0(T))

```

Theses are compile time constant expressions so you may use them even in type-declarations or static initializations as it pleases.

## Pitfall Number Three: signedness

Don’t mix signed and unsigned types in comparisons. The promotion rules are complicated and do not only involve the signedness but also whether or not the type is `long`. In particular, the resulting real comparison may be a comparison of signed values or of unsigned values. Don’t rely on that.

**When comparing values of different signedness, use a cast to promote one of them the signedness of the other.**

This makes your intentions explicit and easier to follow.

BTW, the correct signed type for `size_t` is `ssize_t`, not `ptrdiff_t`.

## Pitfall Number Four: `char` arithmetic

Signedness of `char` is simply a mess, don’t rely on it.

**When you need a \`small’ integer type don’t use `char` but use `[u]int8_t`.**

Use `char` as the base type for strings, and `char*` as a generic pointer type whenever you have to do *byte arithmetic* on pointers. All other uses of `char` should be banned.

## Pitfall Number Five: file offsets

File offsets (and also block sizes) are yet another value that are independent from pointer or integer width. You may well have a 32 bit system (maximal addressable space *4GiB*) with a 64bit file system: think of a large disk partition or of a mount over NFS. The correct type to access file offsets is `off_t`, which is a *signed* type. Usually, there are predefined macros that force your system to use the 64 bit variant, if it is available.

## Pitfall Number Six: pointer conversions and differences

Generally it is a bad idea to convert a pointer to an integer. It really is. Think of it, twice. If it is unavoidable, `int` is certainly the wrong type. The reason for that is that the most convenient integer type of the system may be of a certain width x and the width of pointers may be just another value y.

**Don’t suppose that pointers and integers have the same width**

Use the correct types to transform a pointer to an integer. These have predefined names, too, `intptr_t` and `uintptr_t`, if they exist. If they don’t exist on your system, well really don’t do it. They know why they don’t provide it. On one system these might be `signed` and `unsigned` on another `long` and `ulong`.

When you do pointer arithmetic, use `ptrdiff_t` for differences between pointers. This is guaranteed to have the correct width. But… even then there might be extreme cases where you have an overflow. If you assume for a moment that pointers are 32 bit and you take the difference of one pointer on the stack (often numbers of the form *0xFFFF….*) and another pointer deep down that difference might need the highest bit to code the number and thus if `ptrdiff_t` is also 32 bit wide, the number might overflow. Be carefull.
