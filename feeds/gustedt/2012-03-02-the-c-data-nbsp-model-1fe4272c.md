---
title: the C data&nbsp;model
url: https://gustedt.wordpress.com/2012/03/02/the-c-data-model/
published: "2012-03-02T02:52:54Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1375
---

# the C data&nbsp;model

For beginners in C the different terms that C applies to data items are often confusing. There are variables, literals, integer constants, compound literals, expressions, lvalues, rvalues… In this post I will try to put a bit of systematic into this and at the end move towards a simplified model.

First of all any sort of data in C usually has a **type** and a **value**. For example the literal `0` has the type `signed int`, `0u` is `unsigned int` and `0.0` is `double`. But other kind of data may have different properties, e.g variables may be modifiable and may be scoped, some data has an address in memory and some doesn’t, etc.

In summary data items in C can have the following properties.

- has a type or not
- has a name or not
- has an indeterminate value or not
- has a representation
- is a trap or not
- has a lifetime
- has an address or not
- is mutable or not
- is persistent or not

Additionally to the properties of data items an identifier that refers to such data usually as has *scope* of validity.

In the following I will review these different properties and then I will give an overview table of different specifications with respect to these properties.

## Types

All data that we usually declare or define in C is typed. As mentioned above literals such as `0` or `0.0` have a type that is inferred from their syntax, variables are declared with a primitive type (such as `double` or `_Bool`) or with a composite type ( `struct`, `union` or array).

A notable exception from that rule are objects that we allocate through `malloc` or similar functions. At the start, such an object has an indeterminate value (that is random contents) and no type. It only receives a type when first interpret through a pointer of some base type other than void.

```
void * p = malloc(sizeof(double)); // *p is illegal, the object has neither type nor determined value
double * d = p;                    // *d is of type double, but might have an invalid value
*d = 35.7;                         // *d now holds the double value 35.7

```

The type that an object has is not necessarily unique, C allows to interpret the same object differently according to the access to the object that is chosen. The following `union` type allows to interpret the object that sits behind the identifier `x` either as a `double` or as an `uint64_t`:

```
union {
  double a;
  uint64_t i;
} x = { .a = 13.5 };

```

Initially, the object is interpreted as `double` and initialized with the value `13.0`. Whenever we’d use `x.a` in our program we’d see the actual contents of `x` interpreted as a `double`. But if we use `x.i` we see the same bit pattern but then interpreted as an `uint64_t`, that is a 64 bit wide unsigned integer type. I have chosen `uint64_t` for the example, because that type has the special property that all bit patterns have a valid interpretation. E.g on my machine the above definition gives me the hexadecimal value `0x402B000000000000U` for `x.i`.

## Names

The identifiers that refer to variables that we declare in a C program are the typical example for a name. But there are other names that can occur in a program and that *in fine* can refer to a data item:

- *Variables* declare an *object* by associating a name and a type.
- *Enumeration constants* associate a name and an integer *value*. (C wouldn’t speak of an object here.)
- *Designators* refer to a subobject in a composite type as we have seen for the `.a` member in the `x` union variable above.
- *Preprocessor macros* refer to textual substitutions that are replaced early during compilation. They may (or may not) refer to an object or value. A typically example is `errno` the error status code of the C library that evaluates to some thread specific black magic.

## Values and representations

These to are not the same thing, but are easily confounded. A *value* is an abstract entity that is used in a program. Variables, objects, literals, enumeration constants all usually have values that determine the state of the so-called abstract machine. That is at a given moment a program execution of a C program is determined by the program itself and the values that are associated to all of its declared or allocated objects, literals etc.

This association is a *semantic* interpretation, that is it gives a *sense* (don’t take this too philosophical, though) to a data representation inside an object. As we have seen in the above example for `x` the same representation of the object to which `x` refers is different according to the view-point that we chose to interpret it:

- the bit pattern stored in the object would be `0100 0000 0010 1011 0000 ...`
- the value when look at through `x.a` is `13.5`
- the value when looked at through `x.i` is `4623789442425946112`

So both values have the same representation. The value comes to a representation by interpreting it through a specific *type*.

The same value can also have different representations. The abstract value `-1` can typically be encoded as

- `1111 1111` for a signed char
- `1111 1111 1111 1111` for a 16 bit integer
- `1011 1111 111 0000 ...` for a `double`.

## Indeterminate values and traps

As we already have seen with the malloc example above not all objects in C hold a determined value. Any object that is not initialized can hold any value that the platform sees fit, in C jargon this is usually just an *unspecific* value. In many cases for memory backed objects this will just be any random value that had been previously stored in the same location. This does not necessary lead to *undefined behavior* (UB), but only means that your program may go down branches that you never expected.

Memory backed objects can have indeterminate values when they come to live and aren’t properly initialized. There are two types of objects that behave such: objects obtained through `malloc` (etc) and automatic variables that are declared in function scope. In particular, `auto` variables are a common pitfall. Especially having an uninitialized pointer point to a random address in the process memory is generally a very bad idea. There is only one way to avoid these problems, *always* initialize variables, and be it just with the default initializer like that:

```
double * d = 0;
totoType toto = { 0 };

```

Things are getting worse if the bit pattern that is stored in an object doesn’t have a valid interpretation within the type through which the object is looked at. Such representations are called *trap* representations. Accessing a variable that is in such a state *is* undefined behavior, if your program does that all bets are off. Anything can happen. If your lucky just your program crashes, or just your computer, or the program publishes your credit card number in a newsgroup (don’t worry nobody reads that anymore) or on a social networking site.

Trap representations are rare on modern architectures. I don’t know of any of the modern processors that has such a trap for integer types. The classical example is the integer value with sign bit set and all other bits unset. In two’s complement this may either be the minimum value of the type or a trap. By the standard there is only one integer type that is guaranteed not to have a trap representation: `unsigned char`. That is with a construct like

```
union overl {
  double a;
  unsigned char bytes[sizeof(double)];
};

```

we will always be able to inspect the individual bytes of any data type.

## Lifetime

The lifetime of a data item can be:

- The whole time of the program execution. In this category are variables in file scope (“globals”), `static` variables, simple literals, string literals, some compound literals and enumeration constants.
- Bound to the execution of a specific block or statement of the program. These are all block local variables, function parameters, `for` loop variables and most compound literals.
- Bound to the execution of a specific thread of execution. These are objects declared with the `_Thread_local` storage specifier.
- From an explicit allocation (usually through `malloc`) to an explicit deallocation (usually with `free`) or up to the program termination.

Compound literals are temporary objects that are not bound to an identifier, but through a somewhat crude syntax, namely something like `(unsigned){ 1u }`, a cast followed by an initializer. As a general rule they have the lifetime of the *execution* of the block in which they are defined, but there is one extra: if the compound literal is also `const` qualified, it can either be realized as a local variable with the lifetime of the current block, or as globally accessible object that has a lifetime of the entire program execution. E.g in

```
if ((&(double const){ 1.0 } == &(double const){ 1.0 })) {
  // do this
} else {
  // do that
}

```

the two compound literals may or may not have the same address.

## Addresses

A data item may or may not have an address. There are much more sorts of items that *don’t* have an address associated with them

- simple literals such as `0`, `0.0` or `'a'`
- enumeration constants
- return values of functions
- values of casts or other operators
- variables declared with the `register` storage class

For these applying the address of operator `&` is an error (“constraint violation”) that must be detected by the compiler.

The later case of `register` variables has the property of still being modifiable (in general). Also our variable `x` from above could have been declared as `register`. So also the fact of having several values (interpretations) of the same representation, has nothing to do with the property (or not) that an object has an address and is “stored” somewhere.

Data items that have an address are

- all other variables, declared without, or with auto, static or \_Thread\_local storage class.
- string literals
- compound literals
- objects allocated with `malloc`

## Modifying data items

Some data items may be modified, others not. In is a constraint violation to attempt to modify

- simple literals
- enumeration constants
- a variable that is qualified with `const`

It is undefined behavior to modify

- an object whose original declaration had a `const` qualification, even if the type of the access is not `const` qualified. This holds for objects declared as variables or compound literals.
- string literals

Freely modified may be objects

- an object whose original declaration didn’t have a `const` qualification. This holds for objects declared as variables or compound literals.
- objects allocated dynamically through `malloc` and Co.

## Persistence of values

Generally, the flow of execution of a program follows the path that is specified: statements are executed in sequence, branches ( `if` or `switch`) are taken according to the value of the controlling expressions, functions are called and returned from, etc. There are three notable exceptions from that rule:

- A signal handler can kick into the execution of the program at any time. Examples for such signal handlers are IO handlers. IO might be delivered asynchronously to the process and the process is notified of that. A signal handle kicks in, handles the request and then passes control back to the point where execution was interrupted. This feature has not much to do with multi-tasking or multi-threading OS, such things can happen even in a so-called *freestanding* environments, that is environments with minimal OS support.
- Flow control at a given point can be terminated by a call to `longjmp` and resumed at another point where previously a call to `setjmp` had been placed.
- In an execution that supports threads, the execution of a thread can be halted to pass execution to another thread.

Since such events can affect the data that a process uses, returning from such a disruptive event may have some of the used data items modified without notice. In addition to the above there is are other sources of unexpected changes of the value of objects:

- Objects that represent e.g an IO device such as a terminal or sensor can change without notice.
- An objects that is accessed through different “names” (identifiers, pointers) can have its value changed through one of the names, without the compiler being able to anticipate the change through the other name. That phenomenon is called aliasing.

These sort of events are considered to be exceptional modifications of the control flow and data of a program. Without stating anything explicitly all data is assumed to be robust with respect to these events, data is usually *assumed* to be persistent. Without stating anything special the compiler can always assume that the value of an object is the last value that had been stored into it.

There is one exception to that rule (therefore the usually above) data that is accessed through a pointer can be aliased through a pointer of the same base type:

```
bool bogus(unsigned * a, unsigned * b) {
  if (*a == *b) *a += 2;
  if (*a == *b) *a += 3;
  return *a < *b;
}

```

Here the compiler can't determine if `a` and `b` point to the same data location or not. E.g at the second comparison, if `a` had been modified `b` might have changed, too. So the compiler *must* reload `b`‘s value from memory.

If we know that both parameters always point to different addresses, this reloading would be superfluous and thus present a loss of efficiency. C has the restrict keyword to give the compiler the promise that we will always call the function with pointers to different objects:

```
bool bogus(unsigned *restrict a, unsigned *restrict b) {
  if (*a == *b) *a += 2;
  if (*a == *b) *a += 3;
  return *a < *b;
}

```

This function now has a different semantic. The second comparison can never be true and thus the whole branch can be optimized out.

For the other exceptional modifications of objects, the compiler will per default simply assume that they cannot occur. If we think/plan/suspect that such an out-of-order modification may happen we have to declare the type through which we access the object with a `volatile` qualifier. That is when we have data that is accessed

- in a signal handler
- in the exception branch after a `setjmp`
- between different threads (without being `_Atomic` or protected by other means such as a mutex)

we have to declare it with `volatile`, such that each modification of the object is properly recorded through a store operation and such that each access to the value is properly done through a load operation.

## Summary

The following table gives an overview of the different properties for different C constructs. There 4 main classes according to the properties of being addressable or not and of being mutable or not. You may observe that these classes are not related to the type of a construct, nor to the application of qualifiers, nor to the scope of validity. Also observe that I have not talked about lvalues or rvalues. Finally this terminology is not really important to understand what is happening.

typednamedindeterminatemay traplifetimeaddressablemutablepersistent`0` or `'a'`yesnononorunnonoyes`enum { AA };`yesyesnonorunnonoyes`(double)A`yesnonodepends on `A`expressionnonoyes`register double const d = 0.0;`yesyesnonoscope entry/exitnonoyes`register double d = 0.0;`yesyesnonoscope entry/exitnoyesyes`register double d;`yesyesyesyes (UB)scope entry/exitnoyesyes`register double volatile d = 0.0;`yesyesnonoscope entry/exitnoyesno`auto double const d = 0.0;`yesyesnonoscope entry/exityesnoyes`(double const){ 0.0 }`yesnononorun or scope entry/exityesnoyes`static double const d = 0.0;`yesyesnonorunyesnoyes`"ABC"`yesnononorunyesnoyes`(double){ 0.0 }`yesnononoscope entry/exityesyesyes`auto double d = 0.0;`yesyesnonoscope entry/exityesyesyes`auto double d;`yesyesyesyesscope entry/exityesyesyes`auto double volatile d = 0.0;`yesyesnonoscope entry/exityesyesno`void * p = malloc(42);`nonoyeslater(de)allocationyesyesyes`static double d;`yesyesnonorunyesyesyes`static double volatile d;`yesyesnonorunyesyesno
