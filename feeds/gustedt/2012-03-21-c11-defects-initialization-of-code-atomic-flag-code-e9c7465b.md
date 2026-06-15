---
title: 'C11 defects: initialization of <code>atomic_flag</code>'
url: https://gustedt.wordpress.com/2012/03/21/c11-defects-initialization-of-atomic_flag/
published: "2012-03-21T16:44:05Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1576
---

# C11 defects: initialization of <code>atomic_flag</code>

With this post I will be starting some random collection of things that I consider being defects of the C11 standard. Perhaps someone of the committee could pick them up or send me a mail to discuss them and eventually formulate a formal defect report.

C11 expresses the intention to have atomic\_flag as a primitive that should allow to emulate all other atomic types and operations, *7.17.8 p3* in a note says:

> The remaining types can be emulated with `atomic_flag`, though with less than ideal properties.

This is a very good concept as far as I can see, but I have one problem to achieve this, initialization. The phrasing for atomic types in general and for atomic\_flag are different, for `atomic_flag`:

> An atomic\_flag that is not explicitly initialized with
>
> `ATOMIC_FLAG_INIT` is initially in an indeterminate state.

for other atomic types:

> An atomic object with automatic storage duration that is not explicitly initialized using `ATOMIC_VAR_INIT` is initially in an indeterminate state; however, the default (zero) initialization for objects with static or thread-local storage duration is guaranteed to produce a valid state.

This has two problems. First, the mentioned valid state (in contrast to the indeterminate state mentioned before) is thus a determinate state, but the value that is stored is not mentioned explicitly. Everything suggests that this would be the same value as for initializing a variable of the underlying base type by `0`. But I think it would have helped to make that explicit.

The second, more difficult problem is how to emulate an atomic type with `atomic_flag` during initialization. Say we emulate with something like

```
struct atomic_int_struct {
  atomic_flag cat;
  int val;
};

```

Then the `ATOMIC_VAR_INIT` macro could simply look like:

```
#define ATOMIC_VAR_INIT(V) { .cat = ATOMIC_FLAG_INIT, .val = (V), }

```

But if I’d have a variable of `atomic_int_struct` with static storage duration

```
struct atomic_int_struct v;

```

is supposed to do the right thing, namely to guarantee that `v` has a valid state at startup, so it should contain a `0` for `.val`, and `.cat` must be in a determinate state. Since all atomic operations should work without problems on `v`, the state of `.cat` can’t be anything else than “clear”.

Now looking into the possible implementations of `atomic_flag` in assembler I didn’t yet meet a processor where the “clear” state would not be naturally represented by an all `0` value. So I guess in any reasonable implementation we would just have

```
#define ATOMIC_FLAG_INIT { 0 }

```

(or some equivalent formulation.)

If this is so, why `ATOMIC_FLAG_INIT` at all? Why not phrasing the same as for the other atomic types

## Proposed change for the initialization of `atomic_flag`

> The default initializer `{ 0 }` may be used to initialize an `atomic_flag` to the clear state. An `atomic_flag` object with automatic storage duration that is not explicitly initialized using `{ 0 }` is initially in an indeterminate state; *however, the default (zero) initialization for objects with static or thread-local storage duration initializes an `atomic_flag` to the clear state.*

## Proposed change for the initialization of atomic objects

> An atomic object with automatic storage duration that is not explicitly initialized using `ATOMIC_VAR_INIT` is initially in an indeterminate state; however, the default (zero) initialization for objects with static or thread-local storage duration is guaranteed to produce a valid state. *The value of that object is the same value as the value of an object of the unqualified type that has been zero initialized.*
