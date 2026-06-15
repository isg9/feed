---
title: struct tags are not identifiers in&nbsp;C++
url: https://gustedt.wordpress.com/2010/09/12/struct-tags-are-not-identifiers-in-c/
published: "2010-09-12T14:05:43Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=347
---

# struct tags are not identifiers in&nbsp;C++

It seems a common mistake to think that a declaration like `struct toto { ... };` in C++ implies the definition of the identifier `toto` as a type. In reality the rule for this is much more subtle than that: it only implies some sort of implicit typedef `struct toto toto;`. When and if in the corresponding scope there is no other identifier of the same name the `toto` refers to the `struct`.

This comes e.g in effect when you try to use the tools from “sys/stat.h” in C++. It defines a function `stat` and a `struct stat` that coexist in the same scope.

This kind of implicit definition is a pitfall when you think of code sharing between C and C++. In the following we will consider four codes that are slight variations of the same idea.

```
/* Compiles in C and C++, output will usually differ for both*/
#include <stdio.h>
static char T = 'a';
int main(int argc, char** argv) {
    struct T { char X[2]; };
    printf("size of T is %zu\n", sizeof(T));
}

```

Here the implicit `typedef` in C++ comes to its full beauty: for C++ the `sizeof` operator refers to the type and not to the variable. Thus the output in C will be `1` (this is a `char` variable, not a character literal) and in C++ it will at least be 2.

```
/* Compiles in C and C++, output will be 1 for both*/
#include <stdio.h>
int main(int argc, char** argv) {
    static char T = 'a';
    struct T { char X[2]; };
    printf("size of T is %zu\n", sizeof(T));
}

```

In this example, the variable `T` in the function scope inhibits the lookup of `T` as a `struct` tag . So the `sizeof` operator will refer to the variable in both languages. Since `sizeof(char)` is always `1` in both cases, this is what will always be printed.

```
/* Compiles in C but not in C++ */
#include <stdio.h>
static char T = 'a';
int main(int argc, char** argv) {
    struct T { char X[2]; };
    printf("size of T is %zu\n", sizeof T);
}

```

Here `T` will be interpreted differently by C and C++ as in the first example. Since the keyword `sizeof` is only valid as a prefix expression before another expression and not in front of a type, this is invalid code in C++.

```
/* Compiles in C and C++, output will be 1 for both*/
#include <stdio.h>
int main(int argc, char** argv) {
    static char T = 'a';
    struct T { char X[2]; };
    printf("size of T is %zu\n", sizeof T);
}

```

This last example is equivalent to the second, only that omitting the parenthesis in the `sizeof` expression ensures that `T` is not taken as a type, here.

Things get even worse. If you define an object with the same name later in the code, the output changes:

```
/* Compiles in C and C++, output will usually differ for both*/
#include <stdio.h>
static char T = 'a';
int main(int argc, char** argv) {
  struct T { char X[2]; };
  printf("size of T is %zu\n", sizeof(T));
  static char T = 'a';
  printf("size of T is %zu\n", sizeof(T));
}

```

In C++ this prints two different values.

[This answer on stackoverflow may give you further insight into this question.](http://stackoverflow.com/questions/1675351/typedef-struct-vs-struct-definitions/1675446#1675446)
