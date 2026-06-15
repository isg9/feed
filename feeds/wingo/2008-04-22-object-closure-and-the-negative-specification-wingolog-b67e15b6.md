---
title: object closure and the negative specification — wingolog
url: https://wingolog.org/archives/2008/04/22/object-closure-and-the-negative-specification
published: "2008-04-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/22/object-closure-and-the-negative-specification
---

# object closure and the negative specification — wingolog

## [object closure and the negative specification](/archives/2008/04/22/object-closure-and-the-negative-specification)

22 April 2008 9:58 PM

- [scheme](/tags/scheme)
- [goops](/tags/goops)
- [kiczales](/tags/kiczales)
- [mop](/tags/mop)
- [guile](/tags/guile)
- [python](/tags/python)
- [object closure](/tags/object%20closure)

Guile-GNOME was the first object-oriented framework that I had ever worked with in Scheme. I came to it with all kinds of bogus ideas, mostly inherited from my C to Python formational trajectory. I'd like to discuss one of those today: the object closure. That is, if an object is code bound up with data, how does the code have access to data?

In C++, object closure is a non-problem. If you have an object, `w`, and you want to access some data associated with it, you dereference the widget structure to reach the member that you need:

```
char *str = w->name;

```

Since the compiler knows the type of `w`, it knows the exact layout of the memory pointed to by `w`. The `->name` dereference compiles into a memory fetch from a fixed offset from the `widget` pointer.

In constrast, data access in Python is computationally expensive. A simple expression like `w.name` must perform the following steps:

1. look up the class of `w` (call it `W`)

2. loop through all of the classes in `W`'s "method resolution order" --- an ordered set of all of `W`'s superclasses --- to see if the class defines a "descriptor" for this property. In some cases, this descriptor might be called to get the value for `name`.

3. find the "dictionary", a hash table, associated with `w`. If the dictionary contains a value for `name`, return that.

4. otherwise if there was a descriptor, call the descriptor to see what to do.

This process is run every time you see a `.` between two letters in python. OK, so `getattr` does have an opcode to itself in CPython's VM instruction set, and the above code is implemented mostly in C (see Objects/object.c:PyObject\_GenericGetAttr). But *that's about as fast as it can possibly get*, because the structure of the Python language definition prohibits any implementation of Python from ever having enough information to implement the direct memory access that is possible in C++.

But, you claim, that's just what you get when you program in a dynamic language! What do you want to do, go back to C++?

**straw man says nay**

"First, do no harm", said a practitioner of another profession. Fundamental data structures should be chosen in such a way that needed optimizations are possible. Constructs such as Python's namespaces-as-dicts actively work against important optimizations, effectively putting an upper bound on how fast code can run.

So for example in the case of the object closure, if we are to permit direct memory access, we should allow data to be allocated at a fixed offset into the object's memory area.

Then, the basic language constructs that associate names with values should be provided in such a way that the compiler can determine what the offset is for each data element.

In dynamic languages, types and methods are defined and redefined at runtime. New object layouts come into being, and methods which operated on layouts of one type will see objects of new types as the program evolves. All of this means that to maintain this direct-access characteristic, the compiler must be present at runtime as well.

So, in my silly `w.name` example, there are two cases: one, in which the `getattr` method is seeing the combination of the class `W` and the slot `name` for the first time, and one in which we have seen this combination already. In the first case, the compiler runs, associating this particular combination of types with a new procedure, newly compiled to perform the direct access corresponding to where the `name` slot is allocated in instances of type `W`. Once this association is established, or looked up as in the second case, we jump into the compiled access procedure.

Note that at this point, we haven't specified what the relationship is between layouts and subclassing. We could further specify that subclasses cannot alter the layout of slots defined by superclasses. Or, we could just leave it as it is, which is what Guile does.

Guile, you say? That slow, interpreted Scheme implementation? Well yes, I recently realized (read: was told) that Guile in fact implements this exact algorithm for dispatching its generic functions. Slot access does indeed compile down to direct access, as far as can be done in a semi-interpreted Scheme, anyway. The equivalent of the \_\_mro\_\_ traversal mentioned in the above description of python's `getattr`, which would be performed by `slot-ref`, is compiled out in Guile's slot accessor generics.

In fact, as a theoretical aside, since Guile dispatches lazily on the exact types of the arguments given to generic functions (and not just the specializer types declared on the individual methods), it can lazily compile methods knowing *exactly* what types they are operating on, with all the possiblities for direct access and avoidance of typechecking that that entails. But this optimization has not yet entered the realm of practice.

**words on concision**

Python did get one thing right, however: objects' code access their data via a single character.

> It is generally true that we tend to believe that the expense of a programming construct is proportional to the amount of writer's cramp that it causes us (by "belief" I mean here an unconscious tendency rather than a fervent conviction). Indeed, this is not a bad psychological principle for language designers to keep in mind. We think of addition as cheap partly because we can notate it with a single character: "+". Even if we believe that a construct is expensive, we will often prefer it to a cheaper one if it will cut our writing effort in half.
>
> Guy Steele, *[Debunking the 'Expensive Procedure Call' Myth, or, Procedure Call Implementations Considered Harmful, or, Lambda: The Ultimate GOTO](http://repository.readscheme.org/ftp/papers/ai-lab-pubs/AIM-443.pdf)* (p.9)

Since starting with Guile, over 5 years ago now, I've struggled a lot with object-oriented notation. The problem has been to achieve that kind of Python-like concision while maintaining schemeliness. I started with the procedural slot access procedures:

```
(slot-ref w 'name)
(slot-set! w 'name "newname")

```

But these procedures are ugly and verbose. Besides that, since they are not implemented as generic functions, they prevent the lazy compilation mentioned above.

GOOPS, Guile's object system, does allow you to define slot accessor generic functions. So when you define the class, you pass the `#:accessor` keyword inside the slot definition:

```
(define-class  ()
  (bar #:init-keyword #:bar #:accessor bar))

(define x (make  #:bar 3))
(bar x) => 3
(set! (bar x) 4)

```

Now for me, typographically, this is pretty good. In addition, it's compilable, as mentioned above, and it's mappable: one can `(map bar list-of-x)`, which compares favorably to the Python equivalent, `[x.name for x in list_of_x]`.

My problem with this solution, however, is its interaction with namespaces and modules. Suppose that your module provides the type, `, or, more to the point, `. If `` has 54 slots, and you define accessors for all of those slots, you have to export 54 more symbols as part of your module's interface.

This heavy "namespace footprint" is partly psychological, and partly real.

It is "only" psychological inasmuch as methods of generic functions do not "occupy" a whole name; they only specify what happens when a procedure is called with particular types of arguments. Thus, if `opacity` is an accessor, it doesn't occlude other procedures named `opacity`, it just specifies what happens when you call `(opacity x)` for certain types of `x`. It does conflict with other types of interface exports however (variables, classes, ...), although classes have their own . \*Global-variables\* do as well, and other kinds of exports are not common. So in theory the footprint is small.

On the other hand, there are real impacts to reading code written in this style. You read the code and think, "where does `bar` come from?" This mental computation is accompanied with machine computation. First, because in a Scheme like Guile that starts from scratch every time it's run, the accessor procedures have to be allocated and initialized every time the program runs. (The alternatives would be an emacs-like `dump` procedure, or R6RS-like separate module compilation.) Second, because the (drastically) increased number of names in the global namespace slows down name resolution.

**lexical accessors**

Recently, I came upon a compromise solution that works well for me: the `with-accessors` macro. For example, to scale the opacity of a window by a ratio, you could do it like this:

```
(define (scale-opacity w ratio)
  (with-accessors (opacity)
    (set! (opacity w)
          (* (opacity w) ratio))))

```

This way you have all of the benefits of accessors, with the added benefit that you (and the compiler) can see lexically where the `opacity` binding comes from.

Well, almost all of the benefits, anyway: for various reasons, for this construct to be implemented with accessors, Guile would need to support subclasses of generic functions, which is does not yet. But the user-level code is correct.

Note that `opacity` works on instances of any type that has an `opacity` slot, not just windows.

Also note that the fact that we allow slots to be allocated in the object's memory area does not prohibit other slot allocations. In the case of ``, the getters and setters for the `opacity` slot actually manipulate the `opacity` GObject property. As you would expect, no memory is allocated for the slot in the Scheme wrapper.

For posterity, here is a defmacro-style definition of with-accessors, for Guile:

```
(define-macro (with-accessors names . body)
  `(let (,@(map (lambda (name)
                  `(,name ,(make-procedure-with-setter
                            (lambda (x) (slot-ref x name))
                            (lambda (x y) (slot-set! x name y)))))
                names))
     ,@body))

```

**final notes**

Interacting with a system with a meta-object protocol has been a real eye-opener for me. Especially interesting has been the interplay between the specification, which specifies the affordances of the object system, and the largely unwritten "negative specification", which is the set of optimizations that the specification hopes to preserve. Interested readers may want to check out Gregor Kiczales' work on meta-object protocols, the canonical work being his "The Art of the Metaobject Protocol". All of Kiczales' work is beautiful, except the aspect-oriented programming stuff.

For completeness, I should mention the [java-dot notation](http://jscheme.sourceforge.net/jscheme/doc/javaprimitives.html), which has been adopted by a number of lispy languages targetting the JVM or the CLR. Although I guess it meshes well with the underlying library systems, I find it to be ugly and non-Schemey.

And regarding Python, lest I be accused of ignoring `__slots__`: the `getattr` lookup process described is the same, even if your class defines `__slots__`. The `__slots__` case is handled by descriptors in step 2. This is [specified in the language definition](http://docs.python.org/ref/slots.html). If slots were not implemented using descriptors, then you would still have to do the search to see if there were descriptors, although some version of the lazy compilation technique could apply.

## related articles

- [class redefinition in guile](/archives/2009/11/09/class-redefinition-in-guile)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)

### 5 responses

1. [Juri Pakaste](http://www.juripakaste.fi/blog/) says:[23 April 2008 7:09 AM](#112fde4af0abb6fb580347a2a26dcac18b82a8cd)

   Yeah. I've never been very comfy with OO code in either CL or Scheme, due to namespace issues and verboseness of sending messages/calling methods. I did some OO code in MzScheme's class.ss recently which is based on messages and those (send ob message) things really cramp my style. On the generic function side, CLOS has the problem of fixing the number of arguments for each generic function which makes you sweat every function name extra hard. Guile thankfully avoids that.

   It's one of the reasons why I occasionally doubt if sexps are the ultimate syntax after all. It seems that more syntaxy languages like Python and Ruby solve this very neatly, with the added benefit of always knowing exactly where a name comes from (unless you're insane and do a from Foo import \*).

2. [James Henstridge](http://blogs.gnome.org/jamesh/) says:[23 April 2008 2:40 PM](#f5f8451d251c90b4cf1c3b287b203871af690063)

   Note that the MRO scan isn't performed for every method in Python: there are fast paths for the special methods where they cache the actual method to be used for each class.

   The downside is that it makes mutating classes more difficult since that can affect method selection in subclasses.

   When the new class system was being developed for Python 2.2, Guido originally made the class dict of new-style classes immutable after creation to allow doing this type of optimisation for all method calls. That decision was a bit unpopular so was reversed before the release (it also didn't handle people updating \_\_bases\_\_ very well either ...).

3. [wingo](http://wingolog.org/) says:[23 April 2008 3:07 PM](#21473b6b10006cc8dc57f8d643adf74a7cd39bfc)

   James: Interesting history, I wasn't aware of that. Thanks for the note.

   I don't know exactly in which cases it runs, but there is a cache invalidation protocol with Guile's generics as well.

   Juri: I occassionaly doubt as well.

   Heh, this reminds me of something:

   > Purification is a continuing process, often institutionalized in the cult of confession, which enforces conformity through guilt and shame evoked by mutual criticism and self-criticism in small groups.
   >
   > Confessions contain varying mixtures of revelation and concealment. As Albert Camus observed, "Authors of confessions write especially to avoid confession, to tell nothing of what they know." Young cult members confessing the sins of their precultic lives may leave out ideas and feelings that they are not aware of or reluctant to discuss, including a continuing identification with their prior existence. Repetitious confession, especially in required meetings, often expresses an arrogance in the name of humility. As Camus wrote: "I practice the profession of penitence to be able to end up as a judge," and, "The more I accuse myself, the more I have a right to judge you."
   >
   > Robert Jay Lifton, [Cult formation](http://www.rickross.com/reference/brainwashing/brainwashing1.html)

   Lambda: the ultimate cult?

4. [wingo](http://wingolog.org/) says:[24 April 2008 3:43 PM](#c4b0392d4b91271c288b3a872e53f170440e7930)

   As long as I'm taking notes in my own comments, the following thread is pretty interesting: [Syntactic sugar and identifier permissivity](http://sources.redhat.com/ml/guile/2000-04/msg00010.html)

5. [Jason Orendorff](http://jorendorff.blogspot.com/) says:[8 January 2009 9:12 PM](#04d1a39e339929077be646fc5a3d0d62cd58837b)

   I don't know if this was the case when you wrote this, but Python now appears to have a method cache, so it doesn't have to dig through the whole mro each time an attribute is got. You could cache more stuff if you wanted to and speed this up more (at the expense of simplicity).

   I don't think phrases like "that's about as fast as it can get" have much bearing on reality here.

   SpiderMonkey, Mozilla's JavaScript engine, has a similarly rich scheme of hooks, getters, and setters, and it has a more elaborate property cache. In the common cases, property lookups very fast indeed. They can even be JITted.

   Anyway, for a theoretical upper bound to matter, we'd have to be near it. Not many dynamic languages are anywhere near the theoretical upper bound of how fast they could go. Python sure isn't. This week it so happens that there's a patch that will speed CPython up by 15-20% on platforms that use GCC: http://bugs.python.org/issue4753 You wouldn't have that kind of thing going on if we were really hugging the limits of what we can get out of the hardware.

   In conclusion, phthbbbbbt.

Comments are closed.
