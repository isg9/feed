---
title: dispatch strategies in dynamic languages — wingolog
url: https://wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages
published: "2008-10-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages
---

# dispatch strategies in dynamic languages — wingolog

## [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)

17 October 2008 4:54 PM

- [goops](/tags/goops)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [lisp](/tags/lisp)
- [object orientation](/tags/object%20orientation)
- [hack](/tags/hack)
- [dispatch](/tags/dispatch)

In a [past dispatch](//wingolog.org/archives/2008/04/22/object-closure-and-the-negative-specification), I described in passing how accessors in Scheme can be efficiently dispatched into direct vector accesses. In summary, if I define a `` type, and make an instance:

```
(define-class  ()
  (x #:init-keyword #:x #:accessor x)
  (y #:init-keyword #:y #:accessor y))

(define p (make  #:x 10 #:y 20))

```

Access to the various bits of what I call the *object closure* of `p` occurs via lazy, type-specific compilation, such that:

```
(y p)

```

internally dispatches to

```
(@slot-ref p 1)

```

So this is all very interesting and such, but *how* do we actually do the dispatch? It turns out that this is a quite interesting problem, one that the JavaScript people have been taking up recently in other contexts. The basic problem is: given a generic operation and a set of parameters with specific types, how do you determine the *exact* procedure to apply to the arguments?

**dynamic dispatch from a moppy perspective**

This problem is known as *dynamic dispatch*. GOOPS, Guile's object system, approaches it from the perspective of the meta-object protocol, the MOP. A MOP is an an extensible, layered protocol that allows the user to replace and extend parts of a system's behaviour, while still allowing the system to maintain performance. (The latter is what I refer to as the "negative specification" -- the set of optimizations that the person writing the specifications doesn't say, but has in the back of her mind.)

The MOP that specifies GOOPS' behavior states that, on an abstract level, the process of applying a generic function to a particular set of arguments is performed by the `apply-generic` procedure, which itself is a generic function:

```
(apply-generic (gf ) args)

```

The default implementation looks something like this:

```
(define-method (apply-generic (gf ) args)
  (let ((methods (compute-applicable-methods gf args)))
    (if methods
	(apply-methods
         gf (sort-applicable-methods gf methods args) args)
	(no-applicable-method gf args))))

```

That is to say, first we figure out which methods actually apply to specific arguments, then sort them, then apply them. Of course, there are many ways that you might want the system to apply the methods. For some methods, you might want to invoke all applicable methods, in order from most to least specific; others, the same but in opposite order; or, in the normal case, simply apply the most specific method, and allow that method to explicitly chain up.

In effect, you want to parameterize the operation of the various components of `apply-generic`, depending on the *type* of the operation. With a MOP, you effect this parameterization by specifying that `compute-applicable-methods`, `apply-methods`, `sort-applicable-methods`, and `no-applicable-method` should themselves be generic functions. They should have default implementations that make sense, but those implementations should be replaceable by the user.

At this point the specification is awesome in its power -- completely general, completely overridable... but what about efficiency? Do you *really* have to go through this entire process just to invoke a method?

**performance in dynamic languages**

> There are four general techniques for speeding up an algorithm: caching, compiling, delaying computation, and indexing.
>
> Peter Norvig, [A
> Retrospective on PAIP](http://norvig.com/Lisp-retro.html), lesson 21.

Just because the process has been specified in a particular way at a high level does not mean that the actual implementation has to perform all of the steps all of the time. My [previous article](//wingolog.org/archives/2008/04/22/object-closure-and-the-negative-specification) showed an example of compilation, illustrating how high-level specifications can compile down to pointer math. In this article, what I'm building to is an exegesis of GOOPS' memoization algorithm, which to me is an amazing piece of work.

The first point to realize is that in the standard case, in which the generic function is an instance of `` and not a subclass thereof, application of a generic function to a set of arguments will map to the invocation of a single implementing method.

Furthermore, the particular method to invoke depends entirely on the concrete types of the arguments at the call site. For example, let's define a cartesian `distance` generic:

```
(define-method (distance (p1 ) (p2 ))
  (define (square z)
    (* z z))
  (sqrt (+ (square (- (x p1) (x p2)))
           (square (- (y p1) (y p2))))))

```

Now, if we invoke this generic on some set of arguments:

```
(distance a b)

```

The particular method to be applied can be determined entirely from the types of *`a`* and *`b`*. Specifically, if both types are subtypes of ``, the above-defined method will apply, and if not, no method that we know about will apply.

Therefore, whatever the result of the dispatch is, we can cache (or *memoize*) that result, and use it the next time, avoiding invocation of the entire protocol of methods.

The only conditions that might invalidate this memoization would be adding other methods to the `distance` generic, or redefining methods in the meta-object protocol itself. Both of these can be detected by the runtime object system, and can then trigger cache invalidation.

**implementation**

So, how to implement this memoization, then? The obvious place to store the cached data is on the generic itself, the same object that has a handle on all of the methods anyway. In the end, dynamic dispatch is always at least one level indirected -- even C++ people can't get around their vmethod table.

Algorithms in computer science are fundmentally about tradeoffs between space and time. If you can improve in one without affecting the other, assuming correctness, then you can say that one algorithm is better than the other. However after the initial optimizations are made, you are left with a tradeoff between the two.

This point is particularly salient in the context of generic functions, some of which might have only one implementing method, while others might have hundreds. How to choose an algorithm that makes the right tradeoff for this wide range in input?

Guile does something really interesting in this regard. First, it recognizes that the real input to the algorithm is the set of types being dispatched, *not* the set of types for which a generic function is specialized.

This is based on the realization that at runtime, a given piece of code will only see a certain, reduced set of types -- perhaps even one set of types.

**linear search**

In the degenerate but common case in which dispatch only sees one or two sets of types, you can do a simple linear search of a vector containing typeset-method pairs, comparing argument types. If this search works, and the vector is short enough, you can dispatch with a few `cmp` instructions.

This observation leads to a simple formulation of the method cache, as a list of a list of classes, with each class list tailed by the particular method implemention.

In Algol-like pseudocode, because this algorithm is indeed implemented in C:

```
def dispatch(types, cache):
    for (i=0; i(The tradeoff between vectors and lists is odd at short lengths; the former requires contiguous blocks of memory and goes through malloc, whereas the latter can take advantage of the efficient uniform allocation and marking infrastructure provided by GC. In this case Guile actually uses a vector of lists, due to access patterns.)hash cacheIn the general case in which we see more than one or two different sets of types, you really want to avoid the linear search, as this code is in the hot path, called every time you dispatch a generic function.For this reason, Guile offers a second strategy, optimistic hashing. There is still a linear search through a cache vector, but instead of starting at index 0, we start at an index determined from hashing all of the types being dispatched.With luck, if we hit the cache, the linear search succeeds after the first check; and if not, the search continues, wrapping around at the end of the vector, and stops with a cache miss if we wrap all the way around.So, the algorithm is about the same, but rotated:def dispatch(types, cache):
    i = h = hash(types) % len(cache)
    do
        entry=cache[i]
        x = types
        while 1:
           if null(x) && null(cdr(entry)): return car(entry)
           if null(x): break
           if car(x) != car(entry): break
           x = cdr(x); entry = cdr(entry)
        i = (i + 1) % len(cache)
    while i != h
    return false
OK, all good. But: how to compute the hash value?One straightforward way to do it would be to associate some random-ish variable with each type object, given that they are indeed first-class objects, and simply add together all of those values. This strategy would not distinguish hash values on argument order, but otherwise would be pretty OK.The standard source for this random-ish number would be the address of the class. There are two problems with this strategy: one, the lowest bits are likely to be the same for all classes, as they are aligned objects, but we can get around that one. The more important problem is that this strategy is likely to produce collisions in the cache vector between different sets of argument types, especially if dispatch sees a dozen or more combinations.Here is where Guile does some wild trickery that I didn't fully appreciate until this morning when following all of the code paths. It associates a vector of eight random integers with each type object, called the "hashset". These values are the possible hash values of the class.Then, when adding a method to the dispatch cache, Guile first tries rehashing with the first element of each type's hashset. If there are no collisions, the cache is saved, along with the index into the hashset. If there are collisions, it continues on, eventually selecting the cache and hashset index with the fewest collisions.This necessitates storing the hashset index on the generic itself, so that dispatch knows how the cache has been hashed.I think this strategy is really neat! Actually I wrote this whole article just to get to here. So if you missed my point and actually are interested, let me know and I can see about explaining better.fizzleoutOnce you know exactly what types are running through your code, it's not just about dispatch -- you can compile your code given those types, as if you were in a static language but with much less work. That's what Psyco does, and I plan on getting around to the same at some point.Of course there are many more applications of this in the dynamic languages community, but since the Fortran family of languages has had such a stranglehold on programmer's minds for so many years, the techniques remain obscure to many.Thankfully, as the years pass, other languages approach Lisp more and more. Here's to 50 years of the Buddhist virus!
```

## related articles

- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [class redefinition in guile](/archives/2009/11/09/class-redefinition-in-guile)

### 3 responses

1. [Keith Rarick](http://xph.us/) says:[17 October 2008 9:03 PM](#80963571c8979abaf6b085a32e737c9dd73ffd90)

   \> In the end, dynamic dispatch is always at least

   \> one level indirected -- even C++ people can't

   \> get around their vmethod table.

   One can actually do better than C++...

   With a polymorphic inline cache, dynamic dispatch can be reduced to a jump followed by a comparison in the common case. Branch prediction reduces the time spent in the comparison to near zero.

   See http://www.avibryant.com/2006/09/ruby-and-strong.html and http://research.sun.com/self/papers/pics.html for more thorough information.

2. [Andy Wingo](http://wingolog.org/) says:[19 October 2008 9:01 PM](#bce91176b038ad7942ceeee8d8d3d4fce10742d1)

   Thanks for the link, Keith! That PIC paper is a real gem, I've been mulling over it for the last couple hours.

   Picking nits, it does seem that with multimethod dispatch you will need as many comparisons as there are arguments.

3. John Leuner says:[26 February 2009 10:06 AM](#1af470dd6bd5536465e84a8951edfe0e2f18a897)

   Efficient method dispatch in PCL

   by Gregor J. Kiczales, Luis H. Rodriguez

   http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.9.1451

Comments are closed.
