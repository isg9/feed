---
title: named constants
url: https://gustedt.wordpress.com/2012/08/17/named-constants/
published: "2012-08-17T11:48:13Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1715
---

# named constants

\[Note: This post is now largely obsolete, I only keep it up for reference. Named constants are new features in C23 ( [latest public draft](_wp_link_placeholder)) by means of the `constexpr` construct, and also the underlying type of an enumeration can now be prescribed.\]

It is now exactly two years that I wrote about the underestimated [`register`](http://gustedt.wordpress.com/2010/08/17/a-common-misconsception-the-register-keyword/ "the register keyword") keyword in C. Today I’d like to tell about an idea that is spinning in my head ever since, that `register` variables in file scope would be a perfect tool to offer a feature that is much missed in C: *named constants of arbitrary data types*.

The whole of it is a bit technical, be warned, but the essence of it will be to declare something like the following in a header file, without creating linkage or “instantiation” problems, and helping us to write readable and debuggable code:

```
register const double PI = 3.14159265358979323846;

typedef struct listElem listElem;
struct listElem { unsigned val; listElem* next; };

register const listElem singleton = { 0 };

Data types that we declare in C often have some common values that should be shared by all users of that type. Most prominent in C is the all zero initialized value that we always can enforce trough a default initializer of a static object, others need detailed initializers:
static listElem const singleton = { 0 };         // &amp;lt;- initializer could be omitted
static listElem const singleOne = { .val = 1 };

There are two ways to deal with such const qualified objects in file scope. If they are implemented as above with static storage class, the may result in one object per compilation unit and the compiler might throw spooky warnings on us if we define them in a header file but don’t use them at all.
Another way is to declare them extern in a header file
extern listElem const singleton;

and to define them in one of the compilation units:
listElem const singleton = { 0 };

This second method has the big disadvantage that the value of the object is not available in the other units that include the declaration. Therefore we may miss some opportunities for the compiler to optimize our code. For example an initialization function for our structure type may be written as
inline
listElem* listElem_init(listElem* el) {
 if (el) *el = singleton;
 return el;
}

If the value of singleOne is known, the assignment could be realized without loading it but only with immediates. And listElem_init itself could then much easier be inlined where it is called.
C currently has no real support for global constants for arbitrary data types, not even for all the standard arithmetic types: all integer constants are at least as large as int. For scalar types a cast can be used to produce a value expression:
#define myVeritableFalse ((_Bool)+0)
#define myVeritableTrue ((_Bool)+1)
#define HELLO ((char const*const)&amp;quot;hello&amp;quot;)

Or some predefined macro for the special case of complex types:
#define ORIGIN CMPLXF(0, 0)

The only named constants that can serve as genuine constant expressions as C understands it are of type int and are declared through a declaration of an enumeration:
enum { uchar_upper = ((unsigned)UCHAR_MAX) + 1, };

All this, to just define a constant:

define a new type, the unnamed enumeration type
define a constant in that enumeration
this constant isn’t even of that enumeration type, but an int.

For composite types, in particular structure types, the only way to fabric a value expression that is not an assignable lvalue and such that the address can’t be taken is to first define an object for the desired type and to feed it into an expression such that the result is an rvalue. The only operator that can return a composite type is the assignment operator:
#define SINGLETON ((listElem){ .val = 0, next = 0, } = (listElem const){ .val = 0, next = 0, })

Such expression can be hard to read for a human and a debugger; we have to use two compound literals to create a simple rvalue, one that may be const-qualified where the other can’t. The biggest disadvantage is that such an expression is not suitable for initializers for objects with static storage duration. We’d have to define two different macros, one for the initializer and one for the constant expression:
#define SINGLETON_INIT { .val = 0, next = 0, }
#define SINGLETON_MOD_LVALUE (listElem){ .val = 0, next = 0, }
#define SINGLETON_CONST_LVALUE (listElem const){ .val = 0, next = 0, }
#define SINGLETON (SINGLETON_MOD_LVALUE = SINGLETON_CONST_LVALUE)

In block scope, however, there is a construct that can be used to declare immutable values of which no address can be taken: const-qualified objects with register storage class. All the above could be given in block scope with functionally equivalent definitions:
register const listElem singleton = { 0 };
register const listElem singleOne = { .val = 1 };
register const _Bool myVeritableFalse = 0;
register const _Bool myVeritableTrue = 1;
register const char *const HELLO = &amp;quot;hello&amp;quot;;
register const float _Complex ORIGIN = CMPLXF(0, 0);
register const int uchar_upper = (unsigned)UCHAR_MAX + 1;
register const listElem SINGLETON = { .val = 0, next = 0, };

The idea of this proposal is that there is no apparent reason that these register definitions couldn’t be allowed in file scope.
Proposed modification
The aim of this proposal is to introduce named constants, that are values that are referred through an identifier, by means const-qualified objects with register storage class. Since this construct already exists in block scope, only two features must be introduced to make this concept suitable for the intended use:

Allow the definition of const-qualified objects with register storage class in file scope.
Allow the usage of const-qualified objects with register storage class in constant expressions.

I will not go into the details of the changes to the standard that would be needed to achieve that. You can find a technical proposal through the P99 pages.
Discussion
Validity
Clearly the above changes do not invalidate any valid program under the current standard. They only assure that some formerly invalid programs become valid.
For register objects in block scope, the only semantic change would be that some of these objects now would be considered to be constant expressions, and thus be allowed in some contexts where they weren’t before. In particular, some array bounds of block scope arrays might now become constant and thus an array that before was a VLA might now be an ordinary array. Such a shift only enables more flexibility (by allowing initializers) and extends in some cases the life time of an array.
Register objects in file scope were not permitted before. So here there will be no change for existing conforming programs.
The impact on the allowed grammar is minimal. The only new case is the definition of an identifier in file scope, where now also storage class specifier register may occur, but this only if the type is const qualified in addition.
For the semantic of constant expressions this adds named constants followed by designators to the possibilities.
For the semantic of integer constant expressions this adds some expressions of integer type that contain named constants followed by designators to the possibilities that correspond to fields of integer type.
For the semantic of integer constant expressions or arithmetic constant expressions this adds some expressions of integer type that contain named constants followed by designators to the possibilities that correspond to fields of integer type.
This doesn’t add new types of address constants since the address of register objects can’t be taken.
New file scope objects
The main new feature that is added by this proposal are a restricted class of register objects in file scope, namely those that have a const-qualified type. Since not much of the rest of the description of file scope objects is changed, such const qualified objects would have the following properties:

They have external linkage.
They have of static storage duration.
Taking the address of such an object would be a constraint violation.
Such an object is not a modifiable lvalue.
Since the proposal requires that all declarations are also definitions, there can actually only exactly one such definition.
If the definition has no initializer the object is default initialized just as other objects of static storage duration.
If there is an initializer, it must only contain constant expressions as partial initializer expressions and for designators.

Modifying an object that is defined with const-qualified type would always be undefined behavior, but if the object would be register such a change could slip in when the program is executed. With this proposal we would even have a stronger property, namely that a change could only happen through a constraint violation (and thus requires a compile time diagnostic):
Because of 4. it can never be on the left side of an assignment operator or the operand of an increment operator. A cast never leads to an lvalue. Property 3. ensures that the lvalue can never be accessed through a different type.
Property 1. is desirable to oblige users of this feature to use the identifier that corresponds to a named constant consistently. It doesn’t oblige and implementation to offer a symbol in the sense of the linker program of the platform, but it provides the possibility to do so.
Possible realizations
Treat named constants similar to objects with internal linkage
As mentioned in the modified footnote 121) the simplest possible realization of named constants in file scope is to treat them the same as static const qualified objects. If the compiler is not able to optimize all uses of that object into immediates, this realization may have the disadvantage that each compilation unit gets its own copy of this object.
Such a realization would never lead to unspecific behavior. The only unspecific behavior that several realizations as objects with internal linkage would imply, is that the addresses of these objects would differ. Since the address of such an object can’t be taken, the properties of having the same address (or not) would not be observable.
A test macro code in P99, P99_CONSTANT that just declares a static const qualified object and assures that the compiler cannot issue useless warnings about an unused identifier shows good results: with optimization enabled, gcc and clang both are able to avoid the allocation of the object and use only its value where appropriate. Obviously, such replacement macro cannot test if the address of such an object is taken, but hopefully the implementation of this feature for block scope register objects could easily be reused.
Treat named constants similar to objects with “weak” linkage
Another possibility for a realization is to use a common extension that introduces “weak” linkage, i.e a linkage property that allows for multiple definition of an object in different compilation units. If several such objects occur in different units, the linker then asserts that all are of the same size and arbitrarily chooses one of these for the final program.
This technique is e.g already used by some compiler implementors to realize inline functions. gcc and clang also handle this approach well. But in difference to the earlier approach they always generate an object, so a final program will always contain exactly one copy of each named constant.

```
