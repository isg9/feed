---
title: Don&#8217;t use fake&nbsp;matrices
url: https://gustedt.wordpress.com/2014/09/08/dont-use-fake-matrices/
published: "2014-09-08T13:46:05Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=2169
---

# Don&#8217;t use fake&nbsp;matrices

In two ancient posts I have talked about arrays in modern C, [“don’t be afraid of variably modified types”](http://gustedt.wordpress.com/2011/01/09/dont-be-afraid-of-variably-modified-types/ "don’t be afraid of variably modified types") and  [“VLA as function arguments”](http://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/ "VLA as function arguments"). Still there seem to be a lot of people that, perhaps just by bad habit, that prefer to use “fake matrices” instead of real matrices in C. Unfortunately among these people are a lot of university teachers that preach that bad parole to their students. I just try to make a list of the advantages of real matrices, here.

First of all, what do I mean by “fake matrices”? These are arrays of pointers that are used to emulate matrices. In the following `N` is a potentially large value:

```
double* fakeMatrix[N] = malloc(N*sizeof(double*));
for (size_t i = 0; i < N; ++i) fakeMatrix[i] = malloc(N*sizeof(double));

for (size_t i = 0; i < N; ++i)
    for (size_t j = 0; j < N; ++j)
         fakeMatrix[i][j] = 0.0;

/* Use fakeMatrix for something */

for (size_t i = 0; i < N; ++i) free(fakeMatrix[i]);
free(fakeMatrix);

```

We see with the initialization statement that `fakeMatrix` can indeed be use as a matrix, but its allocation and deallocation are crude and error prone. Before C99, such difficult code was necessary, as soon as `N` was not a constant but a variable.

Now, here is the equivalent code for real C matrices:

```
double (*realMatrix)[N] = malloc(sizeof(double[N][N]));

for (size_t i = 0; i < N; ++i)
    for (size_t j = 0; j < N; ++j)
         realMatrix[i][j] = 0.0;

/* Use realMatrix for something */

free(realMatrix);

```

## The advantages:

- ### They are conceptually simple

- ### The allocated memory is contiguous:

  - The allocation “algorithm” is efficient, in O(1)
  - Memory access is efficient, since there are no gaps between matrix lines
- ### One simple allocation and deallocation

  - no need to declare auxiliary functions.
- ### The compiler does the calculus of indices.

  - the target of `realMatrix[i][j]` can be addressed in one go without using indirection.
- ### For the fake matrix the idea that all lines of the matrix have the same length is only implicit, the type is underspecified.

- ### The allocation method `malloc(sizeof(double[N][M]))` is clearer

  - compiler even does the computation of the size within the correct type, namely `size_t` even if the expression `N*N` (with `N` wrongly chosen as `int`) would overflow.
