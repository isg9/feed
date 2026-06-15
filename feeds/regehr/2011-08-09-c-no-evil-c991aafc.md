---
title: C No Evil
url: https://blog.regehr.org/archives/574
published: "2011-08-09T20:47:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=574
---

# C No Evil

![](https://blog.regehr.org/wp-content/uploads/2011/08/images.jpg)

Your mortal enemy would like to ship a really great product in a week or two. Towards the goal of maximally delaying this product, you may inject a single C preprocessor definition into one of their header files. What is it? Keep in mind that anything which stops the project from compiling will be rapidly detected and therefore does not meet the goal.

Here are a few examples to get started with (some my own, some from a [Reddit thread](http://www.reddit.com/r/programming/comments/jdew0/i_dont_think_c_gets_enough_credit/) today).

> ```
> #define FALSE 1
> #define FALSE exit()
> #define while if
> #define goto
> #define struct union
> #define assert(x)
> #define volatile
> #define continue break
> #define double int
> #define long short
> #define unsigned signed
> ```

Have fun.

**Update:** Someone on hacker news posted a link to [The Disgruntled Bomb](http://thedailywtf.com/Articles/The-Disgruntled-Bomb.aspx), which is based on the same idea. Nice!
