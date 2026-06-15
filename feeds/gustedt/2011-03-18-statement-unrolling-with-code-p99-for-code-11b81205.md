---
title: Statement unrolling with <code>P99_FOR</code>
url: https://gustedt.wordpress.com/2011/03/18/statement-unrolling-with-p99_for/
published: "2011-03-18T23:00:02Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1008
---

# Statement unrolling with <code>P99_FOR</code>

Specifying code that does same or equivalent things to a list of variables, tokens, items or so is an important source of annoyance. It can easily make code unreadable and difficult to maintain. E.g consider the following

```

if ((b == a) && (c == a) && (d == a) && (e == a) && (f == a)) {
  /* all are equal, do something special */
} else {
  /* handle the base case */
}

```

Understanding what that expression does is difficult in one glance and updating it is error prone. Adding another variable, say `g`, to be tested may be simple but withdrawing `a` is a major operation.

With code unrolling as P99 provides it we may implement a macro `P99_ALLEQ`

```

if (P99_ALLEQ(a, b, c, d, e, f)) {
  /* all are equal, do something special */
} else {
  /* handle the base case */
}

```

Here, adding another variable to the list or deleting any of them is not a big deal.

In fact, the implementation of such a macro with P99 is as simple as this:

```

#define P00_ISIT(WHAT, X, I) (X) WHAT
#define P99_ALLEQ(FIRST, ...) P99_FOR(== (FIRST), P99_NARG(__VA_ARGS__), P00_AND, P00_ISIT, __VA_ARGS__)

```

This uses `P99_FOR` to do a macro iteration. That iterative evaluation will produce code for each element that is passed in the `__VA_ARGS__` list, \` `b, c, d, e, f`‘ in our example.

Each of these elements is given to the `P00_ISIT` macro as a second argument, the first being the first argument to `P99_FOR`. Here that first argument is `== (FIRST)`, in our example this leads to \` `== (a)`‘. So when processing the `b` in the list that leads to \` `(b) == (a)`‘, for `c` that is \` `(c) == (a)`‘ and so on.

The other macro that is in the arguments of `P99_FOR` is `P00_AND`. As you expect this is what is responsible for the `&&` operators between these individual expansions for each list item. In fact it does a bit more than given above in that it adds a lot more parenthesis. The real expansion done by the preprocessor is

```

if ((((((((((b) == (a)) && ((c) == (a)))) && ((d) == (a)))) && ((e) == (a)))) && ((f) == (a)))) {
  /* all are equal, do something special */
} else {
  /* handle the base case */
}

```

After we withdraw `a` from the list of arguments the new macro expansion would read

```

if ((((((((c) == (b)) && ((d) == (b)))) && ((e) == (b)))) && ((f) == (b)))) {
  /* all are equal, do something special */
} else {
  /* handle the base case */
}

```

In the same way as `P99_ALLEQ` we may define a whole bunch of other macros that work in the same spirit.

```

#define P99_MINOF(FIRST, ...) P99_FOR(>= (FIRST), P99_NARG(__VA_ARGS__), P00_AND, P00_ISIT, __VA_ARGS__)
#define P99_INFOF(FIRST, ...) P99_FOR(> (FIRST), P99_NARG(__VA_ARGS__), P00_AND, P00_ISIT, __VA_ARGS__)
#define P99_MAXOF(FIRST, ...) P99_FOR(<= (FIRST), P99_NARG(__VA_ARGS__), P00_AND, P00_ISIT, __VA_ARGS__)
#define P99_SUPOF(FIRST, ...) P99_FOR(< (FIRST), P99_NARG(__VA_ARGS__), P00_AND, P00_ISIT, __VA_ARGS__)
#define P99_ONEOF(FIRST, ...) P99_FOR(== (FIRST), P99_NARG(__VA_ARGS__), P00_OR, P00_ISIT, __VA_ARGS__)

```

Here the first use the same macro `P00_AND` as before, only `P99_ONEOF`, the last, uses `P00_OR` to combine the different parts.

P99\_FOR can also be used to build expressions that are a more complicated.

```
P99_ORDERED(<=, a, b, c, d)

```

should expand to

```
((a) <= (b)) && ((b) <= (c)) && ((c) <= (d))

```

Here you see that in the expanded expression b and c each appear two times. So a simple generalization of the tricks above would not work. We have to go in steps to generate that. First, we define a macro that expands to the inner part of the expression:

```
(b)) && ((b) <= (c)) && ((c)

```

You might already see the pattern. We have parts per list item that look similar to \` `(b)) && ((b)`‘ and the separation is done with the operator \` `<=`‘. This is done by these two macros:

```
#define P00_ORDERED_AND(_0, X, _2) (X)) && ((X)
#define P00_ORDERED_OP(OP, _1, X, Y) X OP Y

```

Observe the unbalanced parentheses in `P00_ORDERED_AND`. These two are combined in

```
#define P00_ORDERED_MID(OP, N, ...)                                        \
P99_FOR(OP, N, P00_ORDERED_OP, P00_ORDERED_AND, P99_SUB(1, N, __VA_ARGS__))

```

it supposes that it obtains the desired length `N` as second argument. It also uses another generic P99 macro, `P99_SUB`, to compute the sublist of the arguments that starts at position `1` and has length `N`. E.g `P00_ORDERED_MID(<=, 2, r, s, t)` will expand to

```
(s)) && ((s) <= (t)) && ((t)

```

Now this is not yet a valid C expression, only the middle part of what we want, but the following cascade of four defines does the trick.

```
#define P99_ORDERED(OP, ...) P00_ORDERED(OP, P99_NARG(__VA_ARGS__), __VA_ARGS__)
#define P00_ORDERED(OP, N, ...)                 \
P99_IF_LT(N, 3)                                 \
(P00_ORDERED2(OP,__VA_ARGS__))                  \
(P00_ORDERED3(OP, P99_PRED(N), __VA_ARGS__))
#define P00_ORDERED2(OP, X, Y) (X) OP (Y)
#define P00_ORDERED3(OP, N, ...)                        \
((P99_SUB(0, 1, __VA_ARGS__))                           \
 OP P00_ORDERED_MID(OP, P99_PRED(N), __VA_ARGS__)       \
 OP (P99_SUB(N, 1, __VA_ARGS__)))

```

First `P99_IF_LT` is used to find if the number of arguments is less than 3. In the case it is less a special case `P00_ORDERED_2` is chosen, namely the one that need no middle part. If there are more arguments macro `P00_ORDERED3` prefixes the middle part with the first element of the list and the operator, and appends the analog for the last element. The result of

```
P99_ORDERED(<=, a, b, c, d)
P99_ORDERED(>, 4, 5)

```

are what we’d expect:

```
((a) <= (b)) && ((b) <= (c)) && ((c) <= (d))
(4) > (5)

```
