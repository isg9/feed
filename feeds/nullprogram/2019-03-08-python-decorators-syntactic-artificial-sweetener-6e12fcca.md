---
title: 'Python Decorators: Syntactic Artificial Sweetener'
url: https://nullprogram.com/blog/2019/03/08/
published: "2019-03-08T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2019/03/08/
---

# Python Decorators: Syntactic Artificial Sweetener

## [Python Decorators: Syntactic Artificial Sweetener](/blog/2019/03/08/)

March 08, 2019

nullprogram.com/blog/2019/03/08/

Python has a feature called *function decorators*. With a little bit of syntax, the behavior of a function or class can be modified in useful ways. Python comes with a few decorators, but most of the useful ones are found in third-party libraries.

[PEP 318](https://www.python.org/dev/peps/pep-0318/) suggests a very simple, but practical decorator called `synchronized`, though it doesn’t provide a concrete example. Consider this function that increments a global counter:

```
counter = 0

def increment():
    global counter
    counter = counter + 1

```

If this function is called from multiple threads, there’s a *race* *condition* — though, at least for CPython, it’s [not a *data*
*race*](https://blog.regehr.org/archives/490) thanks to the Global Interpreter Lock (GIL). Incrementing the counter is not an atomic operation, as illustrated by [its byte
code](/blog/2019/02/24/):

```
 0 LOAD_GLOBAL              0 (counter)
 3 LOAD_CONST               1 (1)
 6 BINARY_ADD
 7 STORE_GLOBAL             0 (counter)
10 LOAD_CONST               0 (None)
13 RETURN_VALUE

```

The variable is loaded, operated upon, and stored. Another thread could be scheduled between any of these instructions and cause an undesired result. It’s easy to see that in practice:

```
from threading import Thread

def worker():
    for i in range(200000):
        increment()

threads = [Thread(target=worker) for _ in range(8)];
for thread in threads:
    thread.start()
for thread in threads:
    thread.join()

print(counter)

```

The increment function is called exactly 1.6 million times, but on my system I get different results on each run:

```
$ python3 example.py
1306205
$ python3 example.py
1162418
$ python3 example.py
1076801

```

I could change the definition of `increment()` to use synchronization, but wouldn’t it be nice if I could just tell Python to synchronize this function? This is where a function decorator shines:

```
from threading import Lock

def synchronized(f):
    lock = Lock()
    def wrapper():
        with lock:
            return f()
    return wrapper

```

The `synchronized` function is a higher order function that accepts a function and returns a function — or, more specifically, a *callable*. The purpose is to wrap and *decorate* the function it’s given. In this case the function is wrapped in a mutual exclusion lock. Note: This implementation is very simple and only works for functions that accept no arguments.

To use it, I just add a single line to `increment`:

```
@synchronized
def increment():
    global counter
    counter = counter + 1

```

With this change my program now always prints 1600000.

### Syntactic “sugar”

Everyone is quick to point out that this is just syntactic sugar, and that you can accomplish this without the `@` syntax. For example, the last definition of `increment` is equivalent to:

```
def increment():
    ...

increment = synchronized(increment)

```

Decorators can also be parameterized. For example, Python’s `functools` module has an `lru_cache` decorator for memoizing a function:

```
@lru_cache(maxsize=32)
def expensive(n):
    ...

```

Which is equivalent to this very direct source transformation:

```
def expensive(n):
    ...

expensive = lru_cache(maxsize=32)(expensive)

```

So what comes after the `@` isn’t just a name. In fact, it *looks* like it can be any kind of expression that evaluates to a function decorator. Or is it?

### Syntactic artificial sweetener

Reality is often disappointing. Let’s try using an “identity” decorator defined using `lambda`. This decorator will accomplish nothing, but it will test if we can decorate a function using a lambda expression.

```
@lambda f: f
def foo():
    pass

```

But Python complains:

```
    @lambda f: f
          ^
SyntaxError: invalid syntax

```

Maybe Python is absolutely literal about the syntax sugar thing, and it’s more like a kind of macro replacement. Let’s try wrapping it in parentheses:

```
@(lambda f: f)
def foo(n):
    pass

```

Nope, same error, but now pointing at the opening parenthesis. Getting desperate now:

```
@[synchronized][0]
def foo():
    pass

```

Again, syntax error. What’s going on?

### Pattern matching

The problem is that the Python language reference doesn’t parse an expression after `@`. It [matches a very specific pattern](https://docs.python.org/3/reference/compound_stmts.html#function-definitions) that just so happens to *look* like a Python expression. It’s not syntactic sugar, it’s syntactic artificial sweetener!

```
ator ::= "@" dotted_name ["(" [argument_list [","]] ")"] NEWLINE

```

In a way, this puts Python in the ranks of PHP 5 and Matlab: two languages with completely screwed up grammars that can only parse specific constructions that the developers had anticipated. For example, in PHP 5 (fixed in PHP 7):

```
function foo() {
    return function() {
        return 0;
    };
}

foo()();

```

That is a syntax error:

```
PHP Parse error:  syntax error, unexpected '(', expecting ',' or ';'

```

Or [in any version of Matlab](/blog/2008/08/29/):

```
    magic(4)(:)

```

That is a syntax error:

```
Unbalanced or unexpected parenthesis or bracket

```

In Python’s defense, this strange, limited syntax is only in a single place rather than everywhere, but I still wonder why it was defined that way.

Update: Clément Pit-Claudel pointed out the explanation in the PEP, which references [a 2004 email by Guido van Rossum](https://mail.python.org/pipermail/python-dev/2004-August/046711.html):

> I have a gut feeling about this one. I’m not sure where it comes from, but I have it. It may be that I want the compiler to be able to recognize certain decorators.
>
> So while it would be quite easy to change the syntax to @test in the future, I’d like to stick with the more restricted form unless a real use case is presented where allowing @test would increase readability. (@foo().bar() doesn’t count because I don’t expect you’ll ever need that).

- [python](/tags/python/)
- [lang](/tags/lang/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Python%20Decorators:%20Syntactic%20Artificial%20Sweetener) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Python+Decorators%3A+Syntactic+Artificial+Sweetener).

« [The CPython Bytecode Compiler is Dumb](/blog/2019/02/24/)

» [An Async / Await Library for Emacs Lisp](/blog/2019/03/10/)
