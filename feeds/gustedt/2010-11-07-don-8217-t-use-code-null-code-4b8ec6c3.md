---
title: Don&#8217;t use <code>NULL</code>
url: https://gustedt.wordpress.com/2010/11/07/dont-use-null/
published: "2010-11-07T23:36:47Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=574
---

# Don&#8217;t use <code>NULL</code>

I always thought that using `NULL` whenever I wanted to assign a *null pointer value* to a pointer was a good thing, but today I learned the contrary.

The standard, 6.3.2.3, says, seemingly innocent

> An integer constant expression with the value `0`, or such an expression cast to type `void*`, is called a null pointer constant.

An *integer constant expression* is something that evaluates at compile time and is of an integer type. Integer types are all the standard integer types that we all know, including plain `char` and `_Bool`, and enumeration types. In particular such expressions may contain casts to such types.

So far this all sounds relatively innocent if you think of the primary use of *null pointer constants* namely to be assigned to pointer variables, but see below.

Then later (7.17) is defined the macro

> `NULL` which expands to an implementation-defined null pointer constant

This means that `NULL` may result in at least 12 different standard integer types and many more expressions that result in a null value in that type:

```
(_Bool)0
(char)0
(unsigned char)0
(signed char)0
(unsigned short)0
(signed short)0
(unsigned)0 == 0U
(signed)0 == 0 == '\0' == enumeration_constant_of_value_0
(unsigned long)0 == 0UL
(signed long)0 == 0L
(unsigned long long)0 == 0ULL
(signed long)0 == 0LL

```

and in

```
(void*)0

```

and for any enumeration type `enum etag` or any `typedef tdef` of integer type in

```
(enum etag)0, (tdef)0.

```

What is important here that most of these expressions are only interpreted as `null pointer constants` when they appear in what some people call a *pointer context* that is in a context where the compiler expects a pointer value. But strictly speaking C, doesn’t know such a thing like a pointer or integer context.

This becomes harmful when you pass arguments to va\_arg functions. Here most of the arguments are just placed on the stack as they are, only small integer types are promoted to `int`. Now consider a harmless function such as `printf` that at some point expects a `void*` pointer:

```
printf("%d (%p) %d", 1, NULL, 2);

```

Looks fine? But it isn’t at all. As we have seen depending on the mood of your compiler provider `NULL` here may be many things: something what most people expect, namely `(void*)0`, but also `0L` or `0LL`. The call to `printf` might simply expect the three arguments on wrong places on the stack and crash your program.

Now people seriously propose the following work around for that

```
printf("%d (%p) %d", 1, (void*)NULL, 2);

```

But wasn’t the only justification of the macro `NULL` just that it should remind us that there is a pointer type expected at a certain place? But this fact is already clear from the cast to `void*`. Why not just use

```
printf("%d (%p) %d", 1, (void*)0, 2);

```

that has the same message?

In summary,

- `NULL` obfuscates an expression that can be either of integer type *or* of pointer type.

- Whenever you want to initialize or assign to a pointer to a null value, use plain `0`, it does serve well.

- Whenever you use a null value in a function call for a parameter that has no proto *type* use `(T*)0`.

Ah, and yes, an enumeration casted `0`‘s should be interpreted as *null pointer constants*:

```
enum awk { a };
void* q = (enum awk)0;

```

*clang* accepts this correctly. *gcc* and *opencc* don’t.
