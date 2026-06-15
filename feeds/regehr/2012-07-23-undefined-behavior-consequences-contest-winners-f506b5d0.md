---
title: Undefined Behavior Consequences Contest Winners
url: https://blog.regehr.org/archives/767
published: "2012-07-23T19:27:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=767
---

# Undefined Behavior Consequences Contest Winners

The [contest that I posted the other day](https://blog.regehr.org/archives/759) received some very nice entries. I decided to pick multiple winners since the best entries illustrate consequences of several kinds of undefined behavior.

First, here’s the runner up, submitted by Octoploid:

```
int main (void) {
  int x = 1;
  while (x) x <<= 1;
  return x;
}

```

This program is undefined by C99, where signed left shifts must not push a 1 bit into or past the sign bit. Recent GCC versions such as r189449 for x86-64 on Linux compile this program into an infinite loop at -O2 and higher. This entry is great because in general, compiler developers have been very reluctant to start enforcing the very strict rules for signed left shift in C99. On the other hand, GCC still performs this optimization when asked to compile in C89 mode, where I don’t think the behavior is undefined. Based on that observation, I filed [a GCC bug report.](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=54027) One of the main GCC developers, Joseph Myers, replied with this:

> Shifts in GCC are supposed to be defined as long as the shift amount is in range, independent of the LHS, so it should not be folding like that. (Although we document in implement-c.texi that this is subject to change for signed left shift, I don’t think changing it would be a particularly good idea.)

I think it is great to see compiler developers taking this kind of a stand against exploiting certain undefined behaviors. Anyway, although the matter has not yet been completely resolved, and the optimization is perfectly valid in C99, it seems that the developers will back out this transformation. So, while I find this example to be very interesting, it does not seem like a contest winner.

# Winner \#1

Amonakov submitted this code:

```
enum {N = 32};
int a[N], pfx[N];
void prefix_sum (void)
{
  int i, accum;
  for (i = 0, accum = a[0]; i < N; i++, accum += a[i])
    pfx[i] = accum;
}

```

This function forms a bit of an argument against the very terse style of for-loop afforded by C which (in my opinion) makes the undefined behavior harder to see than it might otherwise have been. The undefined operation here is reading `a[32]`. Recent versions of GCC, apparently as a consequence, decide that the loop exit test can be removed, giving an infinite loop that triggers a segfault when `i` becomes large enough.

# Winner \#2

Nick Lewycky submitted this code:

```
#include <stdio.h>
#include <stdlib.h>

int main() {
  int *p = (int*)malloc(sizeof(int));
  int *q = (int*)realloc(p, sizeof(int));
  *p = 1;
  *q = 2;
  if (p == q)
    printf("%d %d\n", *p, *q);
}

```

Using a recent version of Clang (r160635 for x86-64 on Linux):

> ```
> $ clang -O realloc.c ; ./a.out
> 1 2
> ```

To see the undefined behavior, you need to read the fine print in the C99 standard, which says:

> The realloc function deallocates the old object pointed to by ptr and returns a pointer to a new object that has the size specified by size.

In other words, it is an error to use a pointer after it has been passed as an argument to `realloc`. This particular bit of language does not appear in the ANSI C standard and the situation there is more ambiguous. However, reading between the lines in A.6.2, I believe we can infer that this code was intended to be undefined in C89 as well.

The man pages for realloc() that I looked at do not mention or imply this caveat. If a major compiler is going to exploit this behavior, the documentation must be updated.

# Conclusion

I chose these winners because in each case there is an element of surprise. In other words, I would not have guessed that the compiler would exploit these particular undefined behaviors, even after I understood the program or program fragment completely. [An entry](http://lwn.net/Articles/278137/) submitted by Pascal Cuoq, which is otherwise very interesting, fails this test—I would have expected a modern C compiler to exploit those undefined behaviors.

**UPDATE**: See Pascal’s response [here](http://blog.frama-c.com/index.php?post/2012/07/24/Results-are-in).
