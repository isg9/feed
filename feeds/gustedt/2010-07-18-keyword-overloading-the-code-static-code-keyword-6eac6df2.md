---
title: 'Keyword overloading: the <code>static</code> keyword'
url: https://gustedt.wordpress.com/2010/07/18/keyword-overloading-the-static-keyword/
published: "2010-07-18T07:46:21Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=184
---

# Keyword overloading: the <code>static</code> keyword

The keyword `static` has seen a wider and wider use in the different versions of C and C++. For the use that it has now, compared to the beginnings of C, it would have been better to use another token for it something in the vein of `variant` or `alternative_declaration`… Here I try to list all the different usages of that keyword and how the introduction of `static` in a declaration chooses an alternative version of such a declaration.

### Linkage specification

In C and C++ unless specified otherwise an object that is defined in file scope, *i.e* outside of any function, has static storage and external linkage. That is, space for the object is reserved at compile time and it is visible by the linker. Other code of other compilation units (.o files) may refer to such an object by its name.

The linkage changes when a declaration is prefixed with `static`: the object becomes *internal* to the compilation unit and is not accessible from other units. Different objects in different units with the same name that are declared `static` may co-exist even if they have different type and size.

C++ deprecates this use of `static` and provides the concept of an anonymous `namespace` as a replacement.

### Storage class specification

In C and C++ unless specified otherwise a variable that is defined in function scope has automatic storage: at run time, when the function is called, memory for the variable is reserved on the execution stack. Each call of the function, in particular if is recursive, has its own new version of the variable.

The storage class changes when a declaration is prefixed with `static`: the object becomes *static*. Space for the variable is reserved at compile time and all calls of the function see the *same* variable, read and write into the same storage location.

### Class member declaration

In C and C++ each instance of a `struct` (and in C++ also for a `class`) has its own copy of each of its declared members.

```
struct toto { double a; };
struct toto A;
struct toto B;

```

Here `A.a` and `B.a` are guaranteed to be two different objects with different location in storage

In C++ this changes when the declaration of the member is prefixed with `static`

`struct toti { static double a; }; struct toti A; struct toti B; Here A.a and B.a are guaranteed to be the same object. Both refer to exactly the same storage location. Arrays as function parameters As one of the rough edges of the C language(s), two declaration that look exactly the same (but for the names), one in the prototype and one as a stack variable, result in the declaration of two different types of variables. void foo(double A[10]) { double B[10]; } Inside the scope of foo, A is pointer to double and B is array of ten elements of type double. Even their sizes computed with sizeof are different. C++ inherited this rule. C99 complicates this matter even further by introducing the keyword static to introduce yet another variant void foo(double A[static 10]) { double B[10]; } this doesn't change the rules on how A and B are seen from the inside, but provides an information to the caller side of how much array elements are expected. C99 specifies that it is responsibility of the caller to give at least as many elements as the array bound specifies. For the moment gcc accepts this new syntax and simply ignores this information, so it is not C99 conforming with respect to that feature.`
