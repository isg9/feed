---
title: assignment and initialization
url: https://gustedt.wordpress.com/2010/12/12/assignment-and-initialization/
published: "2010-12-12T04:45:30Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=637
---

# assignment and initialization

The innocent like character `=` has two different meanings in C, assignment and initialization. Unfortunately this difference is somehow not very pronounced and leads to confusion and inappropriate use. Many hard to track bugs are caused by lack of proper initialization.

Much code could become much easier to read if some simple rules (listed towards the end) would be followed.

## Initialization of variables

Initialization is declared at the point of the instantiation of the corresponding variable. It is either implicit or explicit and depends on the storage class of the variable.

- Variables with static storage, that are variables that are declared at function scope or with `static` inside functions.

  ```
  extern double a;    // declaration of an external symbol
  extern void* p;     // another one
  extern unsigned k;  // a third
  double a;           // instantiation, implicitly initialized to 0.0
  void* p;            // instantiation, implicitly initialized to a null pointer
  unsigned k = 42;    // instantiation, explicitly initialized to hold the value 42U
  static bool what;   // declaration and instantiation, implicitly initialized to false
  static bool that = true; // declaration and instantiation, explicitly initialized

  ```

- variables with `auto` or `register` storage, this are variables that are local to a function

  ```
  double a;         // declaration and instantiation, not initialized
  void* p;          // declaration and instantiation, not initialized
  unsigned k = 42;  // declaration and instantiation, initialized to hold the value 42U
  static bool what; // declaration and instantiation, not initialized
  static bool that = true; // declaration and instantiation, explicitly initialized

  ```

  So here we have a lot of variables that are not initialized. An accidental read access of these variables provokes *undefined behavior*, that is it can eat your hard disk, crash the moon into the earth or just steal all your money from your bank account. Usually compilers will warn about uninitialized variables, but unfortunately not always.

Composite types ( `struct`, `union` and arrays) follow the same rules as the primitive types in the examples above. So in particular, if you want to use such a variable in function scope you should initialize it:

```
unsigned A[5] = { 1, 2, 3, [4] = 4 };             // A[3] is set to 0U
struct { double a; int nl; } s = { .nl = '\n' };  // s.a is set to 0.0

```

So the rules are simple:

- You write the initialization values in curly braces on the right of the initialization. This syntax can even be used for non composite variables.
- *Designated initializers* such as `.nl` or `[0]` above can be used to specify to which field a value corresponds.
- Otherwise, the values are used in declaration order of the fields.
- Fields that are not mentioned explicitly are initialized with the value `0` as if it were a static initialization.

## Catch-all initializers

Two supplementary syntactic rules are foreseen in the standard to ease the use of initializers:

- Initializers of nested classes may be *flattened*. E.g in

  ```
  typedef struct toto toto;
  struct toto { double arr[2]; };
  toto A = { .arr = { [0] = 1.0, [2] = 2.0 } };
  toto B = { 1.0, 2.0 };

  ```

  both declarations for `A` and `B` are valid and declare variables that have exactly the same initial value.

- Initializers for primitive types may also have braces around them. The following is valid:

  ```
  unsigned i = { 1 };
  void const*const pp = { 0 };

  ```

As a special feature from these rules there is a *catch-all inintializer*: `{ 0 }`. The syntax allows to use this for literally the initialization of *any* variable, regardless of its type. So having a good convention to deal with `0` initialized fields is essential for a good API of C data types. This even holds for `union` types, but as always you’d have to be careful with union types, only the first named field is initialized consistently.

In P99 you may use a more visually appealing form of that:

```
double a = P99_INIT;
double A[7] = P99_INIT;

```

Some compiler (gcc) compiler complains when you use the catch-all initializer for nested structures, since he suspects you to have forgotten some braces. In P99 we thus switch this warning off.

## Assignment to variables

Assignment to variables can be done anywhere inside functions. An assignment overwrites an eventual other value that the variable previously held. For composite types, the syntax of assignment is different from initialization, the curly brace syntax from above is not allowed as such and leads to a syntax error. In historical C, variables could only be declared at the beginning of a scope. So there one would perhaps have been tempted to write something like:

```
// file scope
typedef struct thing thing;
struct thing { double a; int nl; };

void func(void) {
 // declaration and instantiation, but uninitialized
 unsigned A[5];
 thing s;
 .
 A = { 1, 2, 3, 0, 4 };  // error
 s = { 0.0, '\n' };             // error
 .
}

```

We will see below how this can be replaced at least partly.

## Initialization by assignment

What you often see in antique C code is something like this

```
void func(void) {
 unsigned A[5];
 size_t i;
 thing s;
 .
 A[0] = 1; A[1] = 2; A[2] = 3; A[3] = 0; A[4] = 4;
 s.a = 0.0; s.nl = '\n';
 .
 for (i = 0; i < n; ++i) {
    .
 }
 .
 // re-use variable i, here
}

```

This has important problems.

- As you can convince yourself easily the initialization of `A` is much error prone. A missing element is quickly overlooked.
- This kind of initialization by assignment is not robust. If in the course of development a field is added to `thing` all initializations of instances become invalid.
- The more or less arbitrary value of the loop variable `i` is taken as the initial value in a completely different section of the code.

## Localization

The first cure that C99 offers for this kind of “assignment” problem is to allow for a declaration of a variable anywhere in a function. We may then just use an initializer instead of an assignment and the problem disappears.

As a special rule we may declare the loop variable of a `for`-loop in the `for`-statement itself.

```
 for (unsigned i = 0; i < n; ++i) {
    .
 }

```

Such a variable is then only visible in that particular loop. The final value of the last iteration cannot drain to other uses of such a variable later in the function.

## Compound literals

If it is not possible to initialize a variable properly, or if we really want to re-assign a *new* value to a variable C99 has another feature called *compound literals*. The syntax is a bit clumsy. In our example above this would read:

```
s = (thing const){ .nl = '\n' };

```

Compound literals can also be defined for arrays, but since we can’t assign to an array as whole in C, this is not of much use, here.

In the context as we use them here, i.e in an assignment and where don’t need an address of such a literal, the type that we use should always be `const` qualified. This leaves more slackness to the compiler on how to carry out this assignment.

Again P99 offers you a little macro that avoids the flicker in the eye:

```
s = P99_LVAL(thing const, .nl = '\n');

```

So `P99_LVAL` receives the type as a first argument and then a list of initializers of arbitrary length. This is really only syntactic sugar for (my) convenience. I find the clash “){” syntax of compound literals and in general the prefix notation of casts particularly difficult to parse.

## Rules for Robustness

- Use the different incarnations of **0** ( `0`, `'\` `0'`, `false`, `0.0`, null pointer) as a default value of variables and fields.
- Use a scope that is as **narrow** as possible to declare variables.
- Use **designated** initializers.
- Use **initializers** instead of assignments wherever this is possible.
- Use **constant compound literals** to assign to a structure as a whole instead of assigning to each field.

By that you make your code robust against

- omission of the initialization of a field
- addition of new fields to structures or changes in the length of an array
- renaming or reordering of structure fields
- drain of values to other sections of your code
