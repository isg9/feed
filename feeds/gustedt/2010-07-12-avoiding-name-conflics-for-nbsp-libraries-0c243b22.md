---
title: Avoiding name conflics for&nbsp;libraries
url: https://gustedt.wordpress.com/2010/07/12/avoiding-name-conflics-for-libraries/
published: "2010-07-12T09:38:58Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=160
---

# Avoiding name conflics for&nbsp;libraries

C programs usually use quite a lot of libraries with a lot of predefined names. These libraries may be coming from different origins, they may be part of the C runtime or your operating system (I’ll assume that this is POSIX), from third parties or be written by yourself or your collaborators. Conflicting names are a nasty thing to track down and may require big changes if they are detected too late.

### Reserved Identifiers

All systems define a whole bunch of names that go far beyond just the keywords of the language. See the links that are provided below if you have a doubt for an individual name. Nevertheless there are systematic patterns that you should avoid for portability to other (future?) systems that you don’t have your hand on.

#### External Symbols in File Scope

All identifiers in file scope that start with an underscore `_` are reserved for the runtime. So don’t use an identifier that could conflict with that rule, in particular

- No globally visible identifier should start with an underscore. This concerns functions, variables, `typedefs` and `enum` constants.
- Avoid names in *tag* space ( `struct` `union` or `enum`) with an underscore. Although they are not in direct conflict with external symbols it might be difficult to find errors because of that.
- Don’t `#define` names starting with an underscore. They might overwrite system symbols.

#### Reserved future keywords and preprocessor macros

Names starting with an underscore followed by a capital letter or a second underscore are reserved for future use as keywords and macros. This is *e.g* what happened during the extension of C from C89 to C99, where the keyword `_Bool` was included. Since this type of name was reserved beforehand, the standards body was free to chose this name for the new Boolean data type. The `typedef` `bool` that most people use is only defined in the standard header file *stdbool.h*. Only when including this header a conflict with that name (that had not been reserved previously) could occur.

Thus, in the inside of functions or as components of `struct` or `union` names with underscores would be ok, in general, as long as they don’t have a capital letter or a second underscore in second position.

#### Reserved future types

POSIX reserves all other names that *end* in `_t` for future use as *types*. Don’t use them, they are ugly anyhow.

### Chose a prefix

If you expect/estimate/dream that the library that you are designing will be use a lot by others stick to a simple naming convention: chose a prefix, like `demo_`. This makes identifier conflicts easy to track and avoids potential conflicts with macro definitions that somebody else might use.

```
  struct demo_coucou {
     unsigned demo_age;
     double demo_weight;
  };

```

Observe that this also uses the prefix for individual fields in the `struct`. If we would just use `age` or `weight` they might conflict with some code that uses this as names of `#define`

### Compatablity of C headers in C++

In C++ `struct` and `union` tags can be used as type identifiers **iff** they are not used in the identifier namespace. Using tag names and identifiers for different kinds of objects/concepts is a bad idea anyhow, so avoid that. I think the best way to do so is to `typedef` all `struct` and `union` types to the same name.

```
typedef struct demo_coucou demo_coucou;

```

For C, this allows you to do a forward declaration of `struct demo_coucou` **and** of `demo_coucou` in just one line. And for C++ this is valid, too, it just makes their convention of the implicit `typedef` explicit. As an additional advantage it hinders you to accidentally declare another identifier of that name yourself, which would much perturb C++ when it sees your declaration.

### Further reading

- [POSIX requirements](http://www.opengroup.org/onlinepubs/009695399/functions/xsh_chap02_02.html)
- [An article on reserved names by the GNU project](http://www.gnu.org/s/libc/manual/html_node/Reserved-Names.html)
