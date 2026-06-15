---
title: Macros versus <code>inline</code> functions
url: https://gustedt.wordpress.com/2010/08/24/macros-versus-inline-functions/
published: "2010-08-24T23:11:39Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=325
---

# Macros versus <code>inline</code> functions

Functions (whether `inline` or not) and macros fulfill different purposes. Their difference should not be seen as ideological as some seem to take it, and what is even more important, they may work nicely together.

Macros are text replacement that is done at compile time and they can do things like

```
#define P99_ISSIGNED(T) ((T)-1 < (T)0)

```

which gives you a compile time expression of whether or not an integral type is signed or not. That is, they are ideally used when the type of an expression is not known (at the definition) and you want to do something about it. On the other hand, the pitfall with macros is that their arguments may be evaluated several times, which is bad because of side effects.

Functions on the other hand are typed, which makes them more strict or, phrased negatively, less flexible. Consider the functions

```
inline
uintmax_t absU(uintmax_t a) {
  return a;
}
inline
uintmax_t absS(uintmax_t a) {
   return (-a < a) ? -a : a;
}

```

The first implements the trivial `abs` function for an unsigned integral type. The second implements it for a signed type. (Yes, it takes an unsigned as argument, this is for purpose.)

We may use these with any integral type. But, the return type will always be of the largest width and there is a certain difficulty on knowing how do choose between the two.

Now with the following macro

```
#define ABS(T, A) ((T)(P99_ISSIGNED(T) ? absS : absU)(A))

```

we have implemented a

- family of functions
- that works for any integral type
- that evaluates its argument only once
- for which any recent and decent compiler will create optimal code

Well, I admit that doing this with abs is a bit artificial, but I hope you get the picture.
