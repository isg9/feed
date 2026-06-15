---
title: partial evaluation in guile — wingolog
url: https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile
published: "2011-10-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile
---

# partial evaluation in guile — wingolog

## [partial evaluation in guile](/archives/2011/10/11/partial-evaluation-in-guile)

11 October 2011 10:01 AM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [javascript](/tags/javascript)
- [compilers](/tags/compilers)
- [optimization](/tags/optimization)
- [inlining](/tags/inlining)
- [partial evaluation](/tags/partial%20evaluation)

Friends, something awesome just happened: [Guile](http://gnu.org/s/guile/) just got itself a respectable inliner.

I have said before on this blog, quoting [commenter](//wingolog.org/archives/2011/07/05/v8-a-tale-of-two-compilers#788347f5d21641a7115ba069f58715848dba9850) [Rémi Forax](http://www.java.net/blogs/forax/), that "inlining is the mother of all optimizations". It is true that inlining opens up space for code motion, constant folding, dead code elimination, and loop optimizations. However, credit might be better laid at the feet of *partial evaluation*, the mother of all inlining algorithms.

Partial evaluation is a source-to-source transformation that takes your program and produces a better one: one in which any computation that can be done at compile-time is already made, leaving only those computations that need to happen at run-time.

For example, the application

```
(+ 2 3)

```

can clearly be evaluated at compile-time. We say that the *source expression* `(+ 2 3)` reduces to `5` via *constant folding*. The result, `5` in this case, is the *residual expression*.

A more complicated example would look like:

```
(let ((string->chars
       (lambda (s)
         (define char-at
           (lambda (n) (string-ref s n)))
         (define len
           (lambda () (string-length s)))
         (let loop ((i 0))
           (if (< i (len))
               (cons (char-at i)
                     (loop (1+ i)))
               '())))))
  (string->chars "yo"))
=> (list #\y #\o)

```

Here when I write `=>`, you should read it as, "residualizes at compile-time to". In this case our input program residualized, at compile-time, to a simple list construction. The loop was totally unrolled, the string-refs folded, and all leaf procedures were inlined.

Neat, eh?

**optimization enables expressiveness**

If the partial evaluator does its job right, the residual program will run faster. However this isn't the real reason that I'm so pleased with it; rather, it's that it lets me write different programs.

You see, I hack on Guile's compiler and VM and all that. When I write code, I know what Guile is going to do with it. Unfortunately, this caused my programs to be uglier than necessary, because I knew that Guile wasn't going to inline some important things for me. I wrote at a lower level of abstraction, because I couldn't trust the compiler.

Now, with the partial evaluator, I'm happy to use helper functions, even higher-order helpers, with the knowledge that Guile will mostly do the right thing. This is particularly important in the context of languages that support syntactic abstraction, like Scheme. If you're a Schemer and haven't seen Kent Dybvig's [Macro Writers' Bill of Rights](http://video.google.com/videoplay?docid=-6899972066795135270) talk ( [slides](http://www.cs.indiana.edu/~chaynes/danfest/dyb.pdf)), do check it out.

Incidentally, there was a sad moment in JSConf.eu a couple weekends ago when Andreas Gal ( [of all people!](http://andreasgal.com/2008/08/22/tracing-the-web/)) indicated that he had to manually inline some functions in PDF.js in order to get adequate speed. More on JavaScript a little later, though.

**about partial evaluation**

A partial evaluator looks a lot like a regular [meta-circular evaluator](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty). It's a recursive function that takes an expression and an environment and yields a value. Guile's partial evaluator, `peval`, builds up lexical environments when it sees `let` and other binding constructs, and tries to propagate copies when it sees lexical references.

Inlining is facilitated by copy-propagation of lambda expressions. Just as the initial value `0` in the example above propagates through the lexical variable `i` to reach `(< i (len))`, `(lambda () (string-length s))` propagates to `len`. Application of a lambda expression reduces to the equivalent of a `let` binding. So for the first iteration of `loop` above, we have:

```
(< i (len))
;; copy propagation
=> (< 0 ((lambda () (string-length s))))
;; beta-reduction
=> (< 0 (string-length s))
;; copy-propagation
=> (< 0 (string-length "yo"))
;; constant-folding
=> (< 0 2)
;; constant-folding
=> #t

```

In this case the condition folded to a constant, so we know at compile-time which branch to take. The second branch is dead, so we eliminate it. The process continues until we finally produce the resulting list.

**down the rabbit hole**

Up to here things are easy: we have a simple, well-typed example that terminates. But to be part of a real-world compiler, a partial evaluator needs to handle real-world code: accessors for mutable data, access to mutable bindings (lexical and global), indefinite recursion, unbound variables, and poorly-typed programs. In addition, a real-world inliner needs to run quickly and avoid producing bloated residual code.

I should take a moment and note that statically-typed, functional languages can avoid a number of these problems, simply by defining them away. It is no wonder that compiler people tend towards early binding. Scheme does exhibit a fair amount of early binding through its use of lexical scope, but it is not a pure functional language. Working on this `peval` was the first time that I wished for immutable pairs in Scheme, as promoted by Racket and R6RS.

Anyway, having mutability in your language isn't so bad. You do miss some optimization opportunities, but that is OK. What is not OK in a production `peval` is spending too much time on an expression.

Guile's solution, following Waddell and Dybvig's excellent [Fast and Effective Procedure Inlining](http://www.cs.indiana.edu/~dyb/papers/inlining.pdf), is to simply count the number of times through the inliner. Each inlining attempt gets a fresh counter, and any work performed within an inlining attempt decrements the counter. When the counter reaches zero, the inlining attempt is aborted, and a call is residualized instead. Since the number of call sites in the program is fixed, and there is a maximum amount of work that will be done at each call site, the resulting algorithm is O(N) in the size of the source program.

Guile's partial evaluator also uses the on-demand, online strategy of Waddell and Dybvig, to allow definitions to be processed in their use contexts. For example, `(cons 1 2)` may be reduced to `#t` when processed as a test, in a conditional. If, after processing the body of a `let`, a binding is unreferenced, then it is processed for effect. Et cetera.

With the effort counter in place, Guile simply tries to inline every call site in the program, knowing that it will bail out if things don't work. It sounds a little crazy, but it works, as Waddell and Dybvig show. The effort counter also serves to limit code growth, though it is a bit crude. In any case I got less than a percent of code growth when optimizing the `psyntax` expander that Guile uses, which is a win in my book.

**caveats**

Partial evaluation can only propagate bindings whose definitions are known. In the case of Guile, then, that restricts inlining to lexical references and primitive references, and notably excludes global references and module imports, or fields of mutable objects. So this does not yet give us cross-module inlining, beyond the [hacks](http://www.gnu.org/software/guile/manual/html_node/Inlinable-Procedures.html) that abuse the macro expander.

This observation has a correlary, in that some languages promote a style of programming that is difficult to analyze. I'm really talking about object-oriented languages here, and the dynamic ones in particular. When you see `o.foo()` in Java, there is at least the possibility that `foo` is a `final` method, so you know you can inline it if you choose to. But in JavaScript if you see `o.foo()`, you don't know *anything*: the set of properties of `o` can and does vary at runtime as people monkey-patch the object `o`, its prototype, or `Object.prototype`. You can even change `o.__proto__` in most JS implementations. Even if you can see that your `o.foo()` call is dominated by a `o.foo = ...` assignment, you *still* don't know anything in ES5, as `o` could have a setter for the `foo` property.

This situation is mitigated in the JavaScript world by a couple of things.

First of all, you don't have to program this way: you can use lexical scoping in a more functional style. Coupled with strict mode, this allows a compiler to see that a call to `foo` can be inlined, as long as `foo` isn't mutated in the source program. That is a property that is cheap to prove statically.

However, as Andreas Gal found out, this isn't something that the mainstream JS implementations do. It is really a shame, and it has a lasting impact on the programs we write.

I even heard a couple people say that in JS, you should avoid deep lexical bindings, because the access time depends on the binding depth. While this is true for current implementations, it is a property of the *implementations* and not of the *language*. Absent `with` and `eval`-introduced bindings, a property that is true in strict-mode code, it is possible to quickly compute the set of free variables for every `function` expression. When the closure is made, instead of grabbing a handle on some sort of nested scope object, a JS implementation can just copy the values of the free variables, and store them in a vector associated with the function code. (You see, a closure is code with data.) Then any accesses to those variables go through the vector instead of the scope.

For assigned variables -- again, a property that can be proven statically -- you put the variables in a fresh "box", and rewrite accesses to those variables to go through that box. Capturing a free variable copies the box instead of its value.

There is nothing new about this technique; Cardelli and Dybvig (and probably others) discovered it independently in the 80s.

This point about closure implementation is related to partial evaluation: people don't complain much about the poor static inliners of JS, because the generally poor closure implementations penalize lexical abstraction. Truly a shame!

\\* \\* \\*

It seems I have digressed. Sorry about that!

I spoke about closures and lexical scope, properties of the JS language that can enable static inlining. The second (and more important) way that JS implementations can support inlining is dynamically. I [trolled](//wingolog.org/archives/2011/06/10/v8-is-faster-than-gcc) about that some months ago. Dynamic inlining is fantastic, when it works, though there are [limiting heuristics](//wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler) (scroll down to "inlining", and note that the exact set of heuristics have changed in the intervening months).

So my last point was about something that Guile does well that JS implementations do poorly, and it's fair that this point should be the reverse. I would like to be able to dynamically inline, but this would mean associating the intermediate representation with Scheme functions. As Guile can compile code ahead-of-time, this means we would have to serialize the IR out to disk, in much the same way as GCC's new link-time optimizer (LTO) does. But I would like to put that off until we change the format of compiled Guile code to be [ELF](//wingolog.org/archives/2011/07/04/guile-2-0-2-building-the-guildhall). Otherwise we run the risk of bloating our runtime memory size.

**try it out**

Guile's partial evaluator was joint work between myself and my fellow Guile maintainer Ludovic Courtès, and was inspired by a [presentation by William Cook](http://www.cs.utexas.edu/~wcook/tutorial/) at [DSL 2011](https://dsl2011.bordeaux.inria.fr/), along with the Waddell and Dybvig's [Fast and Effective Procedure Inlining](http://www.cs.indiana.edu/~dyb/papers/inlining.pdf).

This code is currently only in the development Guile tree, built from git. Barring problems, it will be part of Guile 2.0.3, which should be out in a couple weeks.

You can check out what the optimizer does at the command prompt:

```
>,optimize (let ((x 13)) (* x x))
$1 = 169
>,optimize (let ((x 13)) (* x foo))
$2 = (* 13 foo)

```

Have fun, and send bugs to `bug-guile@gnu.org`.

## related articles

- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [cross-module inlining in guile](/archives/2021/05/13/cross-module-inlining-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)

### 3 responses

1. gasche says:[11 October 2011 1:09 PM](#5540ff6096c60c88ad1c3d5970c4cd2be31f35bb)

   \> Now, with the partial evaluator, I'm happy to use helper

   \> functions, even higher-order helpers, with the knowledge that

   \> Guile will mostly do the right thing.

   We should be careful not to get locked into that mindset. Optimizations are good for performance, but they to be fragile. Well-specified optimizations (such as the concept of 'tail call') can be relied upon, but I wouldn't rely too much on an inliner : what if you add a call to a debug function in a cold path and suddently it doesn't inline anymore?

   We should keep looking for in-language way to express efficient things that also retain modularity, ie. don't force you, like manual inlining, to change the code structure and hurt maintainability.

   Which interesting, undiscovered language features were lying behind your last use of inlining?

   That said, I also think it's good when simple optimizations like partial evaluation can be used to perform what would need, in a less expressive language, complex domain-specific optimisations. The Haskell "repa" library is an example of such trick: if you represent matrices as (int,int)->foo functions, \`transpose\` is trivial to write, and a trivial optimizer can guess than \`transpose(transpose(mat)) == mat\`.

2. [Matt](http://mkg.cc) says:[12 October 2011 5:06 PM](#5affa3f70d811c69ccf32ca9bb8f06ac6755c111)

   Nice article; I was wondering if you could comment on two things:

   Is it trivial to trace a piece of simplified (i.e. partially evaluated) code back to the original code? I'm thinking in particular of error traces, and assigning crashes and compiler errors to particular lines in the not-yet-inlined source code.

   Would Clojure, with its emphasis on immutability, have an easier time of implementing this kind of partial evalutation and inlining? Or in the end will its lack of static typing still make it as hard as it was in Guile?

3. [Matt](http://mkg.cc) says:[13 October 2011 4:58 PM](#19f29a669dc340fd8013b27ca110bbba92bb5b6e)

   Whoops, sorry for the silly question; of course there will be no one-to-one mapping from lines in the simplified code to lines in the original code. I suppose it's possible to maintain a mapping from chunks of inlined code to the chunks of original code that generated it, so that a crash trace can display not just the inlined code that crashed, but also the original code that generated it, with line numbers.

   I wonder then if it might be a good idea to add a keyword to explicitly forbid inlining of a given expression, e.g. (forbid-inline \*expression\*), for debugging purposes?

Comments are closed.
