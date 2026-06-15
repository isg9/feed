---
title: Duck Typing vs. Type Erasure
url: https://nullprogram.com/blog/2014/04/01/
published: "2014-04-01T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2014/04/01/
---

# Duck Typing vs. Type Erasure

## [Duck Typing vs. Type Erasure](/blog/2014/04/01/)

April 01, 2014

nullprogram.com/blog/2014/04/01/

Consider the following C++ class.

```
#include

template <typename T>
struct Caller {
  const T callee_;
  Caller(const T callee) : callee_(callee) {}
  void go() { callee_.call(); }
};

```

Caller can be parameterized to *any* type so long as it has a `call()` method. For example, introduce two types, Foo and Bar.

```
struct Foo {
  void call() const { std::cout << "Foo"; }
};

struct Bar {
  void call() const { std::cout << "Bar"; }
};

int main() {
  Caller<Foo> foo{Foo()};
  Caller<Bar> bar{Bar()};
  foo.go();
  bar.go();
  std::cout << std::endl;
  return 0;
}

```

This code compiles cleanly and, when run, emits “FooBar”. This is an example of *duck typing* — i.e., “If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.” Foo and Bar are unrelated types. They have no common inheritance, but by providing the expected interface, they both work with with Caller. This is a special case of *polymorphism*.

Duck typing is normally only found in dynamically typed languages. Thanks to templates, a statically, strongly typed language like C++ can have duck typing without sacrificing any type safety.

### Java Duck Typing

Let’s try the same thing in Java using generics.

```
class Caller<T> {
    final T callee;
    Caller(T callee) {
        this.callee = callee;
    }
    public void go() {
        callee.call();  // compiler error: cannot find symbol call
    }
}

class Foo {
    public void call() { System.out.print("Foo"); }
}

class Bar {
    public void call() { System.out.print("Bar"); }
}

public class Main {
    public static void main(String args[]) {
        Caller<Foo> f = new Caller<>(new Foo());
        Caller<Bar> b = new Caller<>(new Bar());
        f.go();
        b.go();
        System.out.println();
    }
}

```

The program is practically identical, but this will fail with a compile-time error. This is the result of *type erasure*. Unlike C++’s templates, there will only ever be one compiled version of Caller, and T will become Object. Since Object has no `call()` method, compilation fails. The generic type is only for enabling additional compiler checks later on.

C++ templates behave like a macros, expanded by the compiler once for each different type of applied parameter. The `call` symbol is looked up later, after the type has been fully realized, not when the template is defined.

To fix this, Foo and Bar need a common ancestry. Let’s make this `Callee`.

```
interface Callee {
    void call();
}

```

Caller needs to be redefined such that T is a subclass of Callee.

```
class Caller<T extends Callee> {
    // ...
}

```

This now compiles cleanly because `call()` will be found in `Callee`. Finally, implement Callee.

```
class Foo implements Callee {
    // ...
}

class Bar implements Callee {
    // ...
}

```

This is no longer duck typing, just plain old polymorphism. Type erasure prohibits duck typing in Java (outside of dirty reflection hacks).

### Signals and Slots and Events! Oh My!

Duck typing is useful for implementing the observer pattern without as much boilerplate. A class can participate in the observer pattern without [inheriting from some specialized class](http://raganwald.com/2014/03/31/class-hierarchies-dont-do-that.html) or interface. For example, see [the various signal and slots systems for C++](http://en.wikipedia.org/wiki/Signals_and_slots). In constrast, Java [has an EventListener type for everything](http://docs.oracle.com/javase/7/docs/api/java/util/EventListener.html):

- KeyListener
- MouseListener
- MouseMotionListener
- FocusListener
- ActionListener, etc.

A class concerned with many different kinds of events, such as an event logger, would need to inherit a large number of interfaces.

- [java](/tags/java/)
- [cpp](/tags/cpp/)
- [lang](/tags/lang/)
- [compsci](/tags/compsci/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Duck%20Typing%20vs.%20Type%20Erasure) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Duck+Typing+vs.+Type+Erasure).

« [Northbound 7DRL 2014](/blog/2014/03/31/)

» [Three Dimensions of Type Systems](/blog/2014/04/25/)
