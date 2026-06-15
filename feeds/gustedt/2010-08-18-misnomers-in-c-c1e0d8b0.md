---
title: Misnomers in C
url: https://gustedt.wordpress.com/2010/08/18/misnomers-in-c/
published: "2010-08-18T08:08:40Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=284
---

# Misnomers in C

C has many misnomers concerning keywords. Here I give a table of possible keywords and convenient macro names that might replace them. New keywords in future C standards usually start with an underscore and a capital letter. Macros in new header files may be just convenient names. (For an example take the inclusion of `_Bool` and `bool` into C99).

This is not completely serious, but it might give you an idea of the different concepts. It also should emphasize the fact that none of these keywords is *ignored* by a conforming C compiler. If you have any ideas of other keywords or other possible namings, please let me know.

**keyword/macro****replacement****macro****comment**`and_eq``&=``and_assign`is an assignment not a comparison`const``_Immutable``immutable``_Invariant``invariant``char``_Byte``byte`in combination with `signed` and `unsigned``inline``_Negligible``negligible``or_eq``|=``or_assign`is an assignment not a comparison` register ``_Addressless``addressless`` static ``_Intern``intern`for use in file scope`_Common``common`for use in function scope`_Atleast`atleast`for use in parameter declaration. A similar proposal was rejected by the standards committee`typedef`_Typealias`typealias`Doesn’t define a type`union`_Overlay`overlay`unsigned`_Modulo`modulo`unsigned integers just implement computation modulo a power of 2`xor_eq`^=`xor_assign`is an assignment not a comparison

Edit: I changed

- `once` to `common`, because this reflects better that this is a variable that is common to all invocations of a function.
- `synonym` to `typealias`, to stay closer to the original
