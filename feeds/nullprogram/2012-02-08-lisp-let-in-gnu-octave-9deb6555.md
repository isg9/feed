---
title: Lisp Let in GNU Octave
url: https://nullprogram.com/blog/2012/02/08/
published: "2012-02-08T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/02/08/
---

# Lisp Let in GNU Octave

## [Lisp Let in GNU Octave](/blog/2012/02/08/)

February 08, 2012

nullprogram.com/blog/2012/02/08/

In [BrianScheme](/blog/2011/01/30/), the standard Lisp binding form `let` isn’t a special form. That is, it’s not a hard-coded language feature, or *special form*. It’s built on top of `lambda`. In any lexically-scoped Lisp, the expression,

```
(let ((x 10)
      (y 20))
  (* 10 20))

```

Can also be written as,

```
((lambda (x y)
   (* x y))
 10 20)

```

BrianScheme’s `let` is just a macro that transforms into a lambda expression. This is also what made it so important to implement lambda lifting, to optimize these otherwise-expensive forms.

It’s possible to achieve a similar effect in GNU Octave (but not Matlab, due to [its flawed parser design](/blog/2008/08/29/)). The language permits simple lambda expressions, much like Python.

```
> f = @(x) x + 10;
> f(4)
ans = 14

```

It can be used to create a scope in a language that’s mostly devoid of scope. For example, I can avoid assigning a value to a temporary variable just because I need to use it in two places. This one-liner generates a random 3D unit vector.

```
(@(v) v / norm(v))(randn(1, 3))

```

The anonymous function is called inside the same expression where it’s created. In practice, doing this is stupid. It’s confusing and there’s really nothing to gain by being clever, doing it in one line instead of two. Most importantly, there’s no macro system that can turn this into a new language feature. *However*, I enjoyed using this technique to create a one-liner that generates `n` random unit vectors.

```
n = 1000;
p = (@(v) v ./ repmat(sqrt(sum(abs(v) .^ 2, 2)), 1, 3))(randn(n, 3));

```

Why was I doing this? I was using the Monte Carlo method to double-check my solution to [this math problem](http://godplaysdice.blogspot.com/2011/12/geometric-probability-problem.html):

> What is the average straight line distance between two points on a sphere of radius 1?

I was also demonstrating to [Gavin](http://devrand.org/) that simply choosing two *angles* is insufficient, because the points the angles select are not evenly distributed over the surface of the sphere. I generated this video, where the poles are clearly visible due to the uneven selection by two angles.

This took hours to render with gnuplot! Here are stylized versions: [Dark](https://s3.amazonaws.com/nullprogram/sphere/dark.html) and [Light](https://s3.amazonaws.com/nullprogram/sphere/light.html).

- [octave](/tags/octave/)
- [trick](/tags/trick/)
- [lisp](/tags/lisp/)
- [media](/tags/media/)
- [math](/tags/math/)
- [video](/tags/video/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Lisp%20Let%20in%20GNU%20Octave) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Lisp+Let+in+GNU+Octave).

« [Cartoon Liquid Simulation](/blog/2012/02/03/)

» [Rumor Simulation](/blog/2012/03/09/)
