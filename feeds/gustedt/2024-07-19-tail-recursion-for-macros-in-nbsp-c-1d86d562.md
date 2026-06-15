---
title: Tail recursion for macros in&nbsp;C
url: https://gustedt.wordpress.com/2024/07/19/tail-recursion-for-macros-in-c/
published: "2024-07-19T14:17:44Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4143
---

# Tail recursion for macros in&nbsp;C

In view of the addition of `__VA_OPT__` first to C++ and now to C23, there had been interest in extending the C preprocessor to include recursion. The basic idea would be that `__VA_OPT__` can be used as a test within a macro such as in the following

```
#define AddUp(X, ...) (X __VA_OPT__( + AddUp(__VA_ARGS__)))

```

Indeed, the `__VA_OPT__` clause only inserts its contents if the `...` part of the parameter list received at least one argument. So at a first glance you may think that this should be fine, and the above will replace an invocation `AddUp(a, b, c)` by ( `a + (b + (c)))`. There are two reasons why this doesn’t work well.

The first reason is simple: the above syntax is already valid but does something else than you might expect. In fact, in current a recursive use of a macro never expands but keeps the name of the macro intact. So the result of the macro as written would be ( `a + AddUp(b, c))`, which is not very helpful in this case.

This property for recursive invocation can not be changed easily, because a lot of existing code would become invalid. So to enable real recursion we have to invent a new syntax, for example

```
#define AddUp(X, ...) (X __VA_OPT__( + RECURSIVE(__VA_ARGS__)))

```

[eĿlipsis](https://gustedt.gitlabpages.inria.fr/ellipsis/) now has a feature that emulates this behavior by defining a stack of macros that call each other. It is slightly more complicated than to write the `#define` directive above, but it is easy enough to play around and test it:

[https://gustedt.gitlabpages.inria.fr/ellipsis/ellipsis-recursive\_8dirs.html](https://gustedt.gitlabpages.inria.fr/ellipsis/ellipsis-recursive_8dirs.html)

This approach works well up to a limited size of the argument list, but is doomed to fail when you would try to have it scale to long lists, say a couple of hundred. Indeed, the second reason why recursion over the argument list does not work well in C is the following observation

> C preprocessor recursion over variadic argument lists has quadratic complexity.

This is due to the fact that the C preprocessor always expands arguments. If you’d pass a list of `n` items into `AddUp` the preprocessor sees these `n` items plus `n-1` comma tokens, so in total `2n-1` tokens. The overall work that is done on one recursion level is thus proportional to `2n-1`. If we add that up over all recursion levels, we have a work of `1 + 3 + 5 + ... + 2n-1` which is `n²`.

As this grows quite rapidly with the size of the argument list, the potential of such a recursion is a bit limited.

[eĿlipsis](https://gustedt.gitlabpages.inria.fr/ellipsis/) has a second feature that completely avoids this, tail recursion

[https://gustedt.gitlabpages.inria.fr/ellipsis/extensions.html#tail](https://gustedt.gitlabpages.inria.fr/ellipsis/extensions.html#tail)

It works similar to the above but avoids to expand the `__VA_ARGS__` parameter

```
#define AddUp(X, ...) (X __VA_OPT__( + __VA_TAIL__()))

```

Here the construct `__VA_TAIL__()` stands in for a recursive call of the surrounding macro, only that `__VA_ARGS__` is not expanded but passed through directly to the call. Since the elements of the list are not touched individually, each recursion level has constant complexity and the overall complexity of a recursive call is linear. Also, again because none of the elements of the list is touched individually, we cannot copy the argument list and therefore a second `__VA_TAIL__()` construct would not work and thus results in an empty list.

In fact, `__VA_TAIL__()` also admits another syntax where we place an identifier inside the parenthesis to name the macro that is to be tail called. In the example above we could in fact have used `__VA_TAIL__(AddUp)` with the same result. Here is a more sophisticated example

```
#define LEFT(X, ...) X __VA_OPT__( + (__VA_TAIL__(RIGHT)))
#define RIGHT(X, ...) __VA_OPT__((__VA_TAIL__(LEFT)) + ) X

```

This does indeed shuffle the arguments to the left or to the right according to the parity of their position in the list.

[eĿlipsis](https://gustedt.gitlabpages.inria.fr/ellipsis/) implements this feature quite efficiently: the tail recursion resolves into an iteration over the argument list:

- It collects tokens that appear on the left of the construct in a list. For `AddUp` this would be `(a + (b + ...`
- It collects those that appear on the right on a stack. For `AddOn` this would just consist of closing parenthesis `...))`.

At the end of the iteration these two result lists are then just spliced together to form the overall result.
