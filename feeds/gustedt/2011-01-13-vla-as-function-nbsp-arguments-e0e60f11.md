---
title: VLA as function&nbsp;arguments
url: https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/
published: "2011-01-13T22:05:15Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=778
---

# VLA as function&nbsp;arguments

Now that we aren’t [afraid of variably modified types](https://gustedt.wordpress.com/2011/01/09/dont-be-afraid-of-variably-modified-types/) anymore let us have a look on how these can be used as function arguments.

Therefore we will

- briefly review what an array is,
- [understand why VLA must be local to a function and of `auto` storage duration](https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/#implicitSize)
- [see how arrays are passed to functions (and why this is done so bizarrely)](https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/#arrayToFunc)
- [see how VM objects are passed to functions](https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/#VLAtoFunc)
- [learn how P99 can large ease or life when programming with pointers to VLA](https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/#P99VLA)

An array with known size at compile time can be handled relatively simple by the compiler:

- somewhere it reserves the necessary memory space for the array (static global space or locally, on the stack)
- in places where the array is used it places the address of that memory
- in places where the size of the array is needed it places the compile time known value

## The implicit size information of VLA

Since the size of a VLA is not known at compile or startup time the size information has to be maintained differently for them. A natural idea would be to have a VLA implemented as some sort of compound structure that contains the size plus the array itself. Unfortunately if we want VLA be compatible with “classic” arrays in C this is not possible, because otherwise a definition such as the following would fail:

```
double (*A)[n] = malloc(n * sizeof(double));

```

Here we assign a memory location on the heap to a pointer to VLA. The RHS `malloc` is anything that a C programmer would want to do about such an allocation. If we’d had a hidden “size” field in the VLA, the RHS would have to take that into account

```
double (*A)[n] = malloc(n * sizeof(double) + /* some overhead? */);
double (*A)[n] = malloc(sizeof(double[n]));

```

By long experience such requirement and attention doesn’t even work well for strings and their infamous terminating 0 character, so this would never work reliably with humans as programmers. In particular, since the status of being a VLA is only implicit to the declaration, the first variant would have to be modified if the status of `n` changes: the two declarations for `n`

```
size_t const n = 100;
enum { n = 100 };

```

would make `A` a VLA for the first (needing an overhead) and a “classic” array for the second.

So the size information can’t be held in the same place as the array itself and must be held implicitly at some place that is to the discretion of the compiler. Usually this will be some hidden variable that the compiler places where appropriate and that is evaluated wherever the size information is needed. Such a variable can not be made visible to other compilation units since when we pass the address of VLA to a function in a different unit, this function can not know if this VLA was a global variable, allocated on the stack or on the heap.

There is another complication to operate the size information of a VLA well, a precise time point during the execution of the program where the declaration with all its size expressions is evaluated. Variables with static storage duration don’t have such a point so VLA and generally all VM types can’t be of static storage duration. We the defining execution order of a surrounding function that precisely defines when and of what size a VLA is created, VM type or variable is defined.

## Arrays as function parameters

If we look at a function prototype as the following

```
double sum(double A[100]);

```

When we call such a function

```
double myArr[100];
.
double theSum = sum(myArr);

```

the array, as in all expression but `sizeof`, evaluates to the address of the first element in the array. So in fact an equivalent specification would have been

```
double theSum = sum(&myArr[0]);

```

So all size information even the compile time constant `100` is lost in that calling mechanism. Consequently C silently also changes our function prototype under the hood. In fact the above prototype is equivalent to

```
double sum(double (*A));

```

Now is all size information lost? No for multidimensional arrays things are different, fortunately. If we have a function and a call as here

```
double eigen(double  A[100][100]);
double mySquare[100][100];
double lambda = eigen(mySquare);

```

the evaluation of `mySquare` leads to a pointer to `double[100]`, noted in the crude C type notation as `double(*)[100]`. The above is thus equivalent to

```
double eigen(double  (*A)[100]);
double mySquare[100][100];
double lambda = eigen(&mySquare[0]);

```

So in essence only the first dimension of an array parameter is transformed into a pointer the other type information, including the size, is kept.

In oldish C there only has been the equivalence of

```
double sum(double A[]);
double sum(double (*A));

```

C99 also allows us to specify type modifiers ( `const` and/or `volatile`) for the pointer like this

```
double sum(double A[const]);
double sum(double (*const A));

```

In a syntactically obscure way it has also the possibility to specify that the function can expect a minimum size of elements that are passed via the pointer.

```
double sum(double A[const static 100]);
double sum(double (*const A));

```

Here the array notation is equivalent to the pointer notation only that the function has the right to expect that `*A` has at least 100 elements, meaning in particular that the pointer can’t be 0.

## Passing VM types

When passing a variable `X` of a VM type the implicit size information of `X` must be passed to the called. Since there is no such thing like *run time type information* in C and since generally the implementation of the function will be in another compilation unit that information has to be made explicit. The approach that C takes here is really basic: a parameter declaration is just a declaration as any other, so we have to specify the bounds. Analogously to the above example we may declare

```
double eigen(size_t n, double  A[n][n]);
double mySquare[n][n];
double lambda = eigen(n, mySquare);

```

So here the declaration of parameter `A` of `eigen` evaluates the value `n` that it received as a first parameter. As we know now this is silently rewritten as

```
double eigen(size_t n, double  (*A)[n]);
double mySquare[n][n];
double lambda = eigen(n, &mySquare[0]);

```

As effect, now in the implementation of `eigen` expressions such as `A[4][4]` can be interpreted at runtime and the correct index computations can be provided by the compiler.

The rules for that kind of function *definition* are simple. At the point of definition you can put any expression in the \[\] that is valid at that point. Just as if you had placed the declaration at the beginning of the block of the function:

```
extern size_t factor;
double augen(size_t n, double  (*A)[factor * n]) {
.
}

```

has the same effect as

```
extern size_t factor;
double augen(size_t n, void*secondParameter) {
double  (*A)[factor * n] = secondParameter;
.
}

```

So the way we specified the bounds for the function `eigen` above is just a convention, but probably a quite sane one. Only see that the definition of the function is important for the evaluation of the parameter types not the point of declaration of the prototype. That the expressions for the bound are “unimportant” at the point where the prototype is declared can also be seen from the fact that we may replace them by ‘\*’, there.

```
double augen(size_t n, double  (*A)[*]) ;

```

This ‘\*’ inside the ‘\[\]’ is not the dereference operator but a completely new overloading of the ‘\*’ token.

## P99 for simple interfaces

Making the bounds of a VM typed parameters explicit can be tedious and error prone. P99 tries to help you on that by providing several macros that help you with that. We already have seen `P99_ALEN(A, N)` that with its second argument `N` allows to ask for the bound in the Nth dimension of N. This can be handy where the expressions that leads to these bounds are complicated or where the variables that entered in these expressions might already have changed their value.

P99 also lets you declare, define and call functions easily and can keep track of all the bounds. Let us look at the following inline definition `dotproductFunc` and and macro declaration `dotproduct` that operate on pointers to 1-dimensional VLA:

```
/* header file */
inline
double dotproductFunc(P99_AARG(double, A),
                      P99_AARG(double, B)) {
  register double ret = 0.0;
  P99_DO(size_t, i, 0, P99_ALEN(A, 1))
    ret += (*A)[i] * (*B)[i];
  return ret;
}

#define dotproduct(VA, VB) dotproductFunc(P99_ACALL(VA), P99_ACALL(VB))

/* in just one compilation unit */
extern inline
double dotproductFunc(P99_AARG(double, A),
                      P99_AARG(double, B));

```

The designated user interface is the macro. Whenever there is an expression of the form `dotproduct(myS, myT)` is found in the code it will be replaced by a four argument call that is similar to `dotproductFunc(P99_ALEN(myS, 1), myS, P99_ALEN(myT, 1), myT)`. The function `dotproductFunc` itself is declared with a prototype like

```
double dotproductFunc(size_t const nA, double (*A)[nA],
                      size_t const nB, double (*B)[nB]);

```

In essence this declares two extra parameters for the bounds of the arrays. Inside the function body we don’t need these parameters ( `nA` and `nB` of the example) and may entirely work with the macro `P99_ALEN`.

These macros can even do better than that. 2-dimensional matrix multiplication can be declared with the simple interfaces

```
void multFunc(P99_AARG(double, C, 2),
              P99_AARG(double, A, 2),
              P99_AARG(double, B, 2));
#define mult(CRR, ARR, BRR) multFunc(P99_ACALL(CRR, 2), P99_ACALL(ARR, 2), P99_ACALL(BRR, 2))

```

So here an additional parameter controls the dimension of the VLAs to which the arguments point. The expansion of these macros gets a bit too complicated to present here. If you are really interested in learning on how this works (and not simply want to use it) it is definitively time to read the source.
