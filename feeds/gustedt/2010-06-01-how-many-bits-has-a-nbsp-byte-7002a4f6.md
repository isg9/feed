---
title: How many bits has a&nbsp;byte?
url: https://gustedt.wordpress.com/2010/06/01/how-many-bits-has-a-byte/
published: "2010-06-01T09:51:40Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=7
---

# How many bits has a&nbsp;byte?

I recently stumbled about this seemingly silly question when trying to write a C macro that depends on the width of a type.

So everybody knows the short answer, 8, as is also expressed in the commonly used French word for byte: \`octet’. But surprisingly for me the long answer is more complicated than that: it depends on the historical context, the norms that you apply, and even then you have to dig a lot of text to come to it.

C has a definition of the type `char`, and the language specification basically uses the terms `char` and  byte interchangeably.

Historically, in times there have been platforms with `char` s (and thus bytes) that had a width different from 8, in particular some early computers coded printable characters with only 6 bits and had a word size of 36. And later other constructors found it more convenient to have words of 16 bits to be the least addressable unit. C90 didn’t wanted to exclude such platforms and just stated

> The number of bits in a char is defined in the macro CHAR\_BIT
>
> CHAR\_BIT can be any value but must be at least 8

and even C99 still just states:

> A byte contains CHAR\_BIT bits, and the values of type unsigned char range from 0 to (2^CHAR\_BIT) – 1.

But then, on the page for the include file `stdint.h` it states

> The typedef name int N \_t designates a signed integer type with width N, no  padding  bits,  and  a  two’s-complement representation. Thus, int8\_t denotes a signed integer type with a width of exactly 8 bits.

So far so good, if there is an `int8_t` we can deduce that `sizeof(int8_t) ` must be 1 and `CHAR_BIT` must be 8. But then the POSIX standard says

> The following types are required:
>
> int8\_t
>
> int16\_t
>
> int32\_t
>
> uint8\_t
>
> uint16\_t
>
> uint32\_t

Which forces `CHAR_BIT` to be 8, and basically also implies that at least for small width types the representation must be two’s-complement on any POSIX compatible platform.

**More reading:**

[Some forum discussion](http://www.misra.org.uk/forum/viewtopic.php?f=71&t=973&start=0)

The *POSIX* specification of [`stdint.h`](http://www.opengroup.org/onlinepubs/000095399/basedefs/stdint.h.html) [`limits.h`](http://www.opengroup.org/onlinepubs/009695399/basedefs/limits.h.html)
