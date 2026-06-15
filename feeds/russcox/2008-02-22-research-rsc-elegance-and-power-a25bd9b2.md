---
title: 'research!rsc: Elegance and Power'
url: https://research.swtch.com/power
published: "2008-02-22T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/power
---

# research!rsc: Elegance and Power

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Elegance and Power Russ Cox February 22, 2008 *research.swtch.com/power* Posted on Friday, February 22, 2008.

These are the most elegant programs I've ever seen.

A power series is an infinite-degree polynomial. For example,

![e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \frac{x^4}{4!} + \cdots](https://research.swtch.com/expx.png)

Computation and manipulation of power series has a long and distinguished history as a test of so-called stream processing systems, which manipulate (arbitrarily long finite prefixes of) infinite streams. In the 1970s, Gilles Kahn and David MacQueen used power series as an unpublished test of their [coroutine-based stream-processing system](http://pdos.csail.mit.edu/~rsc/kahn77parallel.pdf). Hal Abelson and Gerald Sussman devote [Section 3.5](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-24.html#%_sec_3.5) of their well-known 1985 textbook [Structure and Interpretation of Computer Programs](http://mitpress.mit.edu/sicp/) to stream processing, although only a single exercise mentions power series. In the late 1980s, Doug McIlroy explored power series processing in the context of Rob Pike's concurrent programming language Newsqueak. He published the work in his 1990 paper “ [Squinting at Power Series](http://swtch.com/~rsc/thread/squint.pdf)” (the Newsqueak interpreter is named *squint*).

Stream processing requires lazy evaluation to keep computations finite. All of these implementations built frameworks for lazy evaluation atop other building blocks: concurrent processes (Kahn and MacQueen, McIlroy) or first-class functions (Abelson and Sussman). It's only natural, then, to consider what the implementations would look like in a language with explicit support and encouragement for lazy evaluation, a language like Haskell. McIlroy revisited the topic in the context of Haskell in his 1998 Functional Pearl, “ [**Power Series, Power Serious**](http://www.cs.dartmouth.edu/~doug/pearl.ps.gz)” (gzipped PS).

Symbolic manipulations can represent a power series as just a list of integer coefficients. Mathematically, this is equivalent to writing the polynormial in Horner form, which repeatedly factors *x* out of the remainder of the polynomial: *F* = *f* 0 \+ *x* *F* 1, or in Haskell, `(f:fs)` (= `f` \+ *x* `fs`).

![](https://research.swtch.com/expx1.png)

In this form, *ex* is `[1, 1, 1%2, 1%6, 1%24, 1%120, ...]`.

As a simple example, McIlroy's addition and multiplication implementations mimic the usual rules:

( *f* *0* \+ *x* *F* *1*) \+ ( *g* *0* \+ *x* *G* *1*) = ( *f* *0* \+ *g* *0*) \+ *x*( *F* *1* \+ *G* *1*)

```
  (f:fs) + (g:gs) = f+g : fs+gs
```

( *f* *0* \+ *x* *F* *1*) ( *g* *0* \+ *x* *G* *1*) = *f* *0* *g* *0* \+ *x*( *f* *0* *G* *1* \+ *g* *0* *F* *1*) \+ *x* 2( *F* *1* *G* *1*)

```
{- (f:fs) * (g:gs) = (f*g : 0) + (0 : f.*gs + g.*fs) + (0 : 0 : fs*gs) -}
   (f:fs) * (g:gs) = f*g : (f.*gs + g.*fs + (0 : fs*gs))
       c .* (f:fs) = c.*f : c.*fs

```

The commented-out definition is a more literal, but less efficient, translation of the equation.

Haskell's pattern matching and operator overloading are responsible for the elegance of the presentation, but it is lazy evaluation that is responsible for the elegance of the computation. The underlying lazy computation mechanism (whether provided directly, as in Haskell, or via other primitives, as in Scheme and Newsqueak) takes care of all the bookkeeping necessary to compute only the terms needed.

McIlroy implements derivative and integral operators as well. Both rely on a helper function to keep track of the leading multiplicative constant:

```
deriv (f:fs) = (deriv1 fs 1) where
    deriv1 (g:gs) n = n*g : (deriv1 gs (n+1))

integral fs = 0 : (int1 fs 1) where
    int1 (g:gs) n = g/n : (int1 gs (n+1))

```

These definitions enable elegant definitions of the power series for *exp*, *sin*, and *cos*:

```
expx = 1 + (integral expx)

sinx = integral cosx
cosx = 1 - (integral sinx)
```

To me, these three equations programs are beautifully elegant, almost magical.

When learning recursion in an introductory programming course (or induction in an introductory math course), it is common to feel like there's no actual work going on: one case just got rewritten in terms of others. The base cases, of course, provide the foundation on which the others are built. Learning to determine which base cases are necessary given the recursive steps is the key to being comfortable with recursion. Only then can you see that the program or proof is structurally sound.

The recursion in the definitions above gives me the same introductary queasiness, because it is hard to see the base cases. Upon closer inspection, the base case is in the expansion of `integral`, which begins with a constant zero term, making it possible to determine the first term in each of those series without further recursion. Having the first term makes it possible to find the second term, and so on.

For me, understanding why they work only enhances the beauty of these programs.

McIlroy continues the theme in his 2000 ICFP paper “ [The Music of Streams](http://www.cs.dartmouth.edu/~doug/music.ps.gz)” (gzipped PS), which contains even more examples but fewer programs.

I've extracted [runnable code from the paper](http://swtch.com/powser.hs). McIlroy maintains [equivalent code](http://www.cs.dartmouth.edu/~doug/powser.html). McIlroy's version uses these shorter versions of `integral` and `deriv`, which will delight Haskell aficionados:

```
int fs = 0 : zipWith (/) fs [1..]
diff (_:ft) = zipWith (*) ft [1..]
```

For the numerical computing aficionados, there is a one-line implementation of power series reversion:

```
revert (0:ft) = rs where rs = 0 : 1/(ft#rs)
```

(For comparison, Donald Knuth devotes Section 4.7 of [Volume 2](http://www-cs-faculty.stanford.edu/~knuth/taocp.html#vol2), about 10 pages, to the manipulation of power series. The equivalent reversion implementation takes about half a page.)

(Comments originally posted via Blogger.)

- [Maht](http://www.blogger.com/profile/01863908675256558774)(February 22, 2008 1:53 PM) You might be interested to read about a guy who had a flair for infinite series :

[Srinivasa Ramanujan](http://en.wikipedia.org/wiki/Ramanujan)

- [Jim Apple](http://www.blogger.com/profile/11080395413026172939)(February 22, 2008 3:46 PM) What is (#)?

- [Russ Cox](http://swtch.com/~rsc/)(February 22, 2008 3:48 PM) (#) is function composition: f#g = f(g). See the paper for details.
