---
title: 'C17: reformulations for&nbsp;atomic_flag'
url: https://gustedt.wordpress.com/2018/08/13/c17-reformulations-for-atomic_flag/
published: "2018-08-13T14:03:27Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3667
---

# C17: reformulations for&nbsp;atomic_flag

In C11, there was a problem with the fact that `atomic_flag` is one of the rare types that is not considered to have a value, but it only has state ( *clear* and *set*) which is changed via function calls. ( `mtx_t` and `cnd_t` are other examples of such types.) There was no established relation between these states and the return value of the `atomic_flag_test_and_set` functions (which is `bool`).

C17 clears that up by prescribing the return values of `atomic_flag_test_and_set`

The C17 description (7.17.8.1 p2) of the `atomic_flag_test_and_set` functions, first sentence now reads:

> Atomically places the atomic flag pointed to by `object` in the set state and returns the value corresponding to the immediately preceding state.

The return value specification (p3) has been changed to:

> The `atomic_flag_test_and_set` functions return the value that corresponds to the state of the atomic flag immediately before the effects. The return value true corresponds to the set state and the return value false corresponds to the clear state.

The description of `atomic_flag_clear` (7.17.8.2, p2), second phrase, has also been clarified as follows:

> Atomically places the atomic flag pointed to by `object` into the clear state.
