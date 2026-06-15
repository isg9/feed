---
title: Iterating Over The Full Range
url: https://blog.regehr.org/archives/536
published: "2011-05-20T20:22:17Z"
feed: regehr
guid: http://blog.regehr.org/?p=536
---

# Iterating Over The Full Range

Every couple of months I need to write code that runs for every value of, for example, a char. This is usually to exhaustively test something so the code should be fast, so I write it in C. At first I always want to write:

> ```
> for (c = CHAR_MIN; c <= CHAR_MAX; c++) {
>   use (c);
> }
> ```

This is very wrong when c has type “char.” Declaring c as “int” is one solution, and here’s another:

> ```
> c = CHAR_MIN;
> do {
>   use (c);
> } while (c++ != CHAR_MAX);
> ```

There is no signed overflow because the char is promoted to int before being incremented. However, neither solution can iterate over all values of an int. It would be possible to cast to unsigned before incrementing (ugly) or use a larger type than int (not supported by all compilers, and not fast on some platforms). Thus I usually end up with something like this:

> ```
> i = INT_MIN;
> while (1) {
>   use (i);
>   if (i == INT_MAX) break;
>   i++;
> }
> ```

But this is super clunky. Anyone have a better idiom for this case?
