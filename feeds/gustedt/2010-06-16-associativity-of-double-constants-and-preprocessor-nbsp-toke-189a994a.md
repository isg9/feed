---
title: 'Associativity of ##, double constants and preprocessor&nbsp;tokens'
url: https://gustedt.wordpress.com/2010/06/16/associativity-of-double-constants-and-preprocessor-tokens/
published: "2010-06-16T14:10:00Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=54
---

# Associativity of ##, double constants and preprocessor&nbsp;tokens

You might one day be confronted to the need to compose `double` constants by using the preprocessor. This is a tricky affair, since already the first naive try like this doesn’t work:

```
#define FRACTIONAL_WRONG(FRAC) .FRAC

```

Why is that so? For the preprocessor the dot and the following parameter are separate tokens. Thus called e.g as `FRACTIONAL_WRONG(1)` something like ‘. 1’ would be produced a stray dot followed by a blank and a number. This is nowhere a valid token sequence for the C compiler. And obviously the following macro, meant to produce a fractional number is wrong for the same reasons:

```
#define FRACTION_WRONG(INT, FRAC) INT.FRAC

```

Ok, we all know, to glue together tokens there is the

`##` operator in the preprocessor. The following actually

works:

```
#define FRACTIONAL(FRAC) . ## FRAC
#define __FRACTION(INT, FRAC) INT ## FRAC
#define _FRACTION(INT, FRAC) __FRACTION(INT, FRAC)
#define FRACTION(INT, FRAC) _FRACTION(INT, FRACTIONAL(FRAC))

/* using it */
#define INTEGERPART 4
#define FRACTIONALPART 01
static double a = FRACTION(INTEGERPART, FRACTIONALPART);

```

But we will see below that this is somehow just be coincidence.

Let us now try to generalize our idea to produce general doubles, including an exponent. One could be tempted to try something like this:

```
#define EXPONENT_WRONG(ESIGN, EXP) E ## ESIGN ## EXP
#define __DOUBLE_WRONG(SIGN, PN, EXP) SIGN PN ## EXP
#define _DOUBLE_WRONG(SIGN, PN, EXP) __DOUBLE_WRONG(SIGN, PN, EXP)
#define DOUBLE_WRONG(SIGN, INT, FRAC, ESIGN, EXP) _DOUBLE_WRONG(SIGN, FRACTION(INT, FRAC), EXPONENT_WRONG(ESIGN, EXP))

```

That is, we would try to first write an analogous macro that composes the exponent and then try to combine the two parts into one global macro. For this seemingly innocent declaration

```
static double b = DOUBLE_WRONG(-, 4, 01, +, 5);

```

My preprocessor says something weird like

```
error_paste.c:27:1: error: pasting "E" and "+" does not give a valid preprocessing token
error_paste.c:27:1: error: pasting "+" and "5" does not give a valid preprocessing token

```

And yours should say something similar, if it is standard compliant. The problem is that a preprocessor token that starts with an alphabetic character may only contain alphanumeric characters (plus underscore). Our example for `FRACTIONAL` only worked, because by chance a \`dot’ followed by numbers is a valid token by itself, namely a floating point number.

A more direct approach would be to have a macro that pastes 6 tokens together

```
#define PASTE6_NOTSOGOOD(a, b, c, d, e, f) a ## b ## c ## d ## e ## f

```

and then hoping that something like the following would work:

```
#define DOUBLE_NOTSOGOOD(SIGN, INT, FRAC, ESIGN, EXP) SIGN PASTE6(INT, ., FRAC, E, ESIGN, EXP)

static double b = DOUBLE_NOTSOGOOD(-, 4, 01, +, 5);

```

An for most preprocessors it would: glued together *from left to right* each intermediate step would always consist of a valid preprocessor token. The actual rules of the preprocessor that allow for this are a bit more complicated, but basically in addition to alphanumeric tokens all starting parts of double constants (without prefix sign) are valid preprocessor tokens. ouff…

… you think. But there is a last subtlety which is the associativity of the `##` operator. It is not specified whether or not it is from left to right. If we fall upon one that does it from right to left, we are screwed. So if we want to be portable, we have to go even further.

```
#define PASTE2(a, b) a ## b
#define _PASTE2(a, b) PASTE2(a, b)
#define PASTE3(a, b, c) _PASTE2(PASTE2(a, b), c)
#define PASTE4(a, b, c, d) _PASTE2(PASTE3(a, b, c), d)
#define PASTE5(a, b, c, d, e) _PASTE2(PASTE4(a, b, c, d), e)
#define PASTE6(a, b, c, d, e, f) _PASTE2(PASTE5(a, b, c, d, e), f)

static double b = PASTE6(4, ., 01, E, +, 7);

```
