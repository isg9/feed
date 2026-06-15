---
title: 'C11 defects: initialization of&nbsp;padding'
url: https://gustedt.wordpress.com/2012/10/24/c11-defects-initialization-of-padding/
published: "2012-10-24T21:55:11Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1789
---

# C11 defects: initialization of&nbsp;padding

The C11 has added an attempt to force compilers to initialize padding of structures and unions under certain circumstances. Unfortunately the situation has become confusing now, since it still foresees that padding can be treated differently from other parts of structures that are not initialized explicitly.

(Let’s concentrate on structures for the following, unions are analogous.)

C11 states in “6.7.9 Initialization”, para 10:

> If an object that has static or thread storage duration is not initialized explicitly, then:
>
> – ….
>
> – if it is an aggregate, every member is initialized (recursively) according to these rules, and any padding is initialized to zero bits;

So this only holds for objects of static storage, at a first view. But then later (para 19) it says in addition:

> If there are fewer initializers in a brace-enclosed list than there are elements or members of an aggregate, or fewer characters in a string literal used to initialize an array of known size than there are elements in the array, the remainder of the aggregate shall be initialized implicitly the same as objects that have static storage duration.

I read here, that padding inside sub-structures that are not initialized explicitly is zero-bit initialized. Let us take an example:

```
struct T {
  char a;
  /* padding b */
  double c;
  /* padding d */
};
static struct T A[2];
static struct T B[2] = { 0 };

```

A structure such as `struct T` has padding “b” between `a` and `c` on most (all?) modern architectures. Doubles are usually aligned on a 4 or 8 bit boundary. I might also have padding “d” after `c`, let us assume that for the sake of the example. Now us look what the standard guarantees for the initialization of `A` and `B`.

For `A` the standard is clear. All named members ( `A[0].a, A[0].c, A[1].a, A[1].c`) are initialized to their default values, `(char)0` and `0.0` in that case. And all bits of the padding “b” and “d” in `A[0]` and `A[1]` are set to zero.

For `B` this is different. Again, all named members ( `B[0].a, B[0].c, B[1].a, B[1].c`) are initialized to their default values. No guarantee is given for the bits of the padding “b” and “d” in `B[0]`, they have an indeterminate value. But for `B[1]` the implicit rule applies so it is initialized as if it where default initialized as whole. So all bits of the padding “b” and “d” in `B[1]` are set to zero.

In summary, some padding in a structure is guaranteed to be zero-bit initialized, some isn’t. I don’t think that such a confusing situation is intentional.

Older versions didn’t make guarantees about the initialization of padding bits at all. So with most existing compilers you’d have to be even more careful, since they don’t implement C11, yet. But AFAIK, clang already does on that behalf.

Also be aware that this only holds for initialization. Padding isn’t necessarily copied on assignment.

### Possible ways to fix this

#### Option 1: Force static storage to zero bits

I see two possible ways to fix this. The first would be to suppose that default initialization for static objects could be something stronger than an explicit initialization by zero. Add a new paragraph after para 8:

> If an object has static or thread storage duration, any padding is initialized to zero bits.

Then, delete all references to padding in the following paragraph (para 9 in current numbering) and change current para 19

> If there are fewer initializers in a brace-enclosed list than there are elements or members of an aggregate, or fewer characters in a string literal used to initialize an array of known size than there are elements in the array, the ***named members*** of the remainder of the aggregate shall be initialized implicitly the same as objects that have static storage duration. ***Padding inside members of objects that are neither of static or thread storage duration and that is hereby default initialized is in an indeterminate state.***

#### Option 2: treat default initialization and initialization by zero the same

I would be much more in favor of not making a difference between these two types of initialization. AFAIK, such a distinction has never been the intent of the standards committee. This could be done by adding a new paragraph after para 8:

> For an object that has is implicitly or explicitly initialized by one of the rules that follow, any padding is initialized to zero bits.
