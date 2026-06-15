---
title: The deprecated attribute in C23 does much more than marking&nbsp;obsolescence
url: https://gustedt.wordpress.com/2024/01/06/the-deprecated-attribute-in-c23-does-much-more-than-marking-obsolescence/
published: "2024-01-06T19:53:00Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4051
---

# The deprecated attribute in C23 does much more than marking&nbsp;obsolescence

You may already have heard that C23 proposes a new syntax feature called attributes and that one of the standard attributes in C23 is called `deprecated`. Actually, we got this whole attribute thing and also some of the standard attributes from C++, where they had been introduced in C++11 (I think). One of the uses of this particular attribute is to mark a given feature as obsolescent. For example in C23 we now have

```
[[deprecated]] char* asctime(const struct tm* timeptr);
[[deprecated]] char* ctime(const time_t* timer);

```

This simply says that user code should not use these functions anymore, and that compilers should issue a diagnostic if they do.

But this is only one of the possible uses of this new feature. First of all, it can be placed on very different kinds of features, functions, types, variables, enumeration members, structure members …

But most importantly it can be put on features that you somehow have to expose in a header, but which are really only part of the internal dealings of your code, something that users just shouldn’t touch because the feature needs careful maintenance or that simply *might* change in the future.

In the C23 edition of Modern C, see [this post](https://gustedt.wordpress.com/2023/12/31/early-access-to-the-c23-edition-of-modern-c/), I just modified a detailed example to use this new feature:

```
typedef struct circular circular;
struct circular {
  size_t  start [[deprecated("privat")]]; /* First element     */
  size_t  len   [[deprecated("privat")]]; /* Number of elements*/
  size_t  cap   [[deprecated("privat")]]; /* Maximum capacity  */
  double* tab   [[deprecated("privat")]]; /* Data array        */
};

```

What you see here, is a structure type that can be used by application code: variables can be declared, the size is known. But none of the members of the structure can be accessed directly without being warned.

Obviously this would not make sense if we didn’t provide functions that deal with this type. Let’s just take two examples:

```
size_t circular_getlength(circular const* c);
circular* circular_init(circular* c, size_t cap);

```

The first is just a simple accessor function that returns the value of the `len` member, the other is a more involved feature that allocates an array for the `tab` member of the appropriate size and initializes the remaining members consistently.

Since these functions are not declared with `[[deprecated]]` they can be used by application code that includes the header, just as any other function.

Now if we want to implement `circular_getlength`, say, for sure we do not want the compiler to bark up the tree when we access the member `len`. Here is how this implementation in the corresponding `.c` source looks like

```
[[deprecated("implementation")]]
size_t circular_getlength(circular const* c) {
  return c ? c->len : 0;
}

```

So if the *implementation* of the function is itself declared to be deprecated an access to a deprecated feature, here `c->len`, will not be diagnosed by the compiler.

> Implementations should use the deprecated attribute to produce a diagnostic message in case the program refers to a name or entity other than to declare it, after a declaration that specifies the attribute, when the reference to the name or entity is not within the context of a related deprecated entity.
>
> C23 clause 6.7.12.4, p6

So here, since our definition sees `deprecated` for itself, the usage of `len` should not be diagnosed.

Modern compilers are already quite up to speed with this feature if you use for example the option `-std=c2x` for `gcc` or `clang`. Only the aspect of not complaining when seeing the implementation is not yet properly implemented in `gcc-13`, yet. But they have a pragma that we can put at the beginning of the `.c` source that fixes this for the moment:

```
#if __GNUC__ > 4 && __GNUC__ <= 13
# pragma GCC diagnostic ignored "-Wdeprecated-declarations"
#endif

```

This just switches the corresponding diagnosis off for this file, but only when it is compiled with a certain range of versions of `gcc`. Code that only includes the header, will not be affected by the pragma and will still diagnose unwanted use of the structure members.

For the curious, `___GNUC__ > 4` is such that this does not trigger for `clang` (which claims to be `gcc-4` for some historical reason and perfectly implements the feature, so there is no need) and `__GNUC__ <= 13` is because I am expecting this to be fixed in the next version of `gcc`.
