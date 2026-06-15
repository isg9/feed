---
title: Inversion of inversion of control — wingolog
url: https://wingolog.org/archives/2005/04/06/100
published: "2005-04-06T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/04/06/100
---

# Inversion of inversion of control — wingolog

## [Inversion of inversion of control](/archives/2005/04/06/100)

6 April 2005 7:00 PM

- [random](/tags/random)
- [python](/tags/python)

**Flumotion**

[Flumotion](http://www.flumotion.net/) 0.1.7 "La Chacha" is out the door. All non-masochists interested in live streaming something should give it a whirl. Free software, free formats, easy to use. Don't [make baby Jesus cry](http://www.google.com/search?q=makes+baby+jesus+cry) by using proprietary formats.

Masochists, on the other hand, can continue using [icecast](http://icecast.org/).

**Inversion²**

```
f = get_file_name()
save_to_file(f)

```

In the good old days, applications were programmed in an imperative, sequential order, where line numbers increased as a process neared completion. So first you get a file name, then you save, and the program remains in control the whole time.

```
w = file_chooser()
connect(w, 'got-file', got_file)
event_loop()

got_file(f) {
    save_file(f)
}

```

Then came the rise of application frameworks and inversion of control. Instead of step-by-step processes, we had to make objects that respond to events, and then yield control to the event loop. This creates theoretical problems with state -- instead of state building up on a stack, cancellable at any time by a return statement, it has to persist in these objects and pending events, especially in the lack of real closures. A step towards messiness, but it does allow us to do many things at the same time.

This problem is particularly acute in [Twisted](http://twistedmatrix.com/), where doing anything interesting will return a Deferred, which is a promise to call back later with the answer. Code becomes a mess of fragmented callbacks, with each able to perform only a fraction of an action before having to defer again. *Menos mal* that python has closures, albeit crippled ones with a stupid lambda syntax.

Incidentally, this is the same difficulty facing most web applications -- after every page you store state somewhere, then recall it back on another page, without the benefit of a stack. The state management is tough to get right, like programming with gotos but without functions.

So, witness the rise of inversion of inversion of control. [Continuation-based web programming frameworks](http://lisp.tech.coop/Web%2FContinuation) all operate under the same idea of dropping out while the result of a computation is unavailable, then continuing once a process has a result. In the meantime, the server does something else, like handling other people's requests.

```
f = wait_for (get_file_name())
save_file(f)

```

In the above pseudo-code, `get_file_name` returns a deferred result, and `wait_for` drops out into the event loop until the result is available. I'd do that in python's generators, but `wait_for` would have to be implemented as a function, and `yield` only returns to its caller. Python supports semantic abstraction (procedures), but not [syntactic abstraction](http://boo.codehaus.org/Syntactic+Macros) (macros).

```
@defer_generator
def save_stuff():
    d = get_file_name()
    yield d
    save_file(d.value())

```

So the deferred results end up being more a part of the procedures, which is OK. `defer_generator` is a decorator, which means `save_stuff = defer_generator(save_stuff)`. It's implemented in [`flumotion.twisted.defer`](https://core.fluendo.com/trac/cgi-bin/trac.cgi/file/flumotion/trunk/flumotion/twisted/defer.py). If an error occurrs, `d.value()` will raise the exception, allowing you to have `try`/ `except` blocks in deferred programming, even if the operation causing the exception was on another machine. Neat.

```
@deferredGenerator
def save_stuff():
    d = waitForDeferred(get_file_name())
    yield d
    save_file(d.getResult())

```

Twisted people had of course thought of this before, but somehow managed to bungle the idea by wrapping the deferred in seventeen characters (15 for `waitForDeferred`, two for parentheses -- see the end of [`twisted.internet.defer`](http://svn.twistedmatrix.com/cvs/trunk/twisted/internet/defer.py?view=markup&root=Twisted)). That and the StudlyCaps twisted style, which continues to bother me for irrational reasons.

**Meta**

Broke down and added a [python category](//wingolog.org/archives/category/python/) on the log. Somehow I find that categorization is a bit dumb though, like the software should do it for me. Other things that bother me: hierarchies of categories, and the random category. Can't quite put my finger on why.

**The folks I dig**

I'm going home at the end of the month! Just for a week or two, and just to get a Spanish stamp on my passport, but hey, it's an excuse for partying. Rock.

## related articles

- [post-harvest moon](/archives/2005/11/13/post-harvest-moon)
- [buggin out](/archives/2005/09/08/buggin-out)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)

Comments are closed.
