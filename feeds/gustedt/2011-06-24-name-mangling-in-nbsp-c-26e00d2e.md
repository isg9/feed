---
title: Name mangling in&nbsp;C
url: https://gustedt.wordpress.com/2011/06/24/name-mangling-in-c/
published: "2011-06-24T19:01:07Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1060
---

# Name mangling in&nbsp;C

Most will know that C++ mangles external names in a compiler specific way such that they encode the types of function parameters and the nesting of `class` es and `namespace` s. People are probably less aware that most C compilers also mangle some names to make them unique inside compilation units.

Namely, most compilers will create “local” symbol for static variables, and since the naming of static variables in function scope is not unique, they have to mangle the names. gcc, e.g, does that by appending a dot and a unique number to the name. Since the dot is not part of any valid identifier, this makes sure that there is no name clash. Let us have a look at two simple variables:

```
  static int f = 0;
  static int e = 42;

```

and the symbols in the compiled object that are visible with `nm`. For gcc I have

> 0000000000000000 d e.1504
>
> 0000000000000004 d f.1503

so this looks something like the original name (the human readable part) followed by a dot and a series number.

(The long numbers are the offsets of the objects in the object file, the “d” says that it is an internal symbol.)

icc does it differently

> 0000000000000004 d e.3.0.2
>
> 0000000000000000 d f.3.0.2

so here we have, as above, the original name and then followed by something that is an identification of the scope inside the compilation unit. Observe that here icc relies on the fact that `e` and `f` are unique in their respective scopes, we come back to that later.

Things become more obscure when the compiler supports universal characters in identifiers. Depending on the environment, the compiler has to mangle such identifiers, since e.g the loader might not support them in the symbol table.

icc chooses something like replacing such a character by \_uXXXX where XXXX is the hex representation of the character. In that case (icc) this results in two subtle compiler bugs. First, this mangling uses valid identifiers that the user is allowed to use, so they may clash for global symbols with identifiers from the same compilation unit or even from other units.

```
int o\u03ba;
int o_u03ba;

```

The symbols that gcc produces for these objects (placed in file scope) are straight forward namely `o_u03ba` and `oκ` which also shows you that the Unicode character with position 03ba is a Greek kappa. In contrast to that icc just has the same external name `o_u03ba` for the two objects. Even if these two are placed in different compilation units, when they are linked together, there is a clash.

Second, icc even mixes up its own internal naming. All goes well as long as the local variables that we declare are `auto`, e.g in the local scope of some function

```
int \u03ba;
int _u03ba;

```

Here `_u03ba` is a valid name inside a function, one leading underscore followed by a non-capitalized letter is allowed. Now as long as we define the variables like that, icc internally distinguished them and all goes fine. But if we declare them as `static` icc’ mangling convention fires back. The internal names are both folded on the name `_u03ba.83.0.1`. Remember that icc needs the “real name” of the variables to distinguish its local `static` s.

The code still behaves somewhat as expected if the compiler can optimize the access to the static location away and hold the values in temporaries. It then crashes at seemingly random points when the optimizer can’t keep track of the value anymore or when the variables are also declared `volatile`.

```
#include <stdio.h>
int o\u03ba;
int o_u03ba;
int main(int argc, char*argv[]) {
  printf("addresses %p and %p\n", (void*)&o\u03ba, (void*)&o_u03ba);
  static int volatile \u03ba;
  static int volatile _u03ba;
  \u03ba = !!argc;
  _u03ba = !argc;
  printf("values %d and %d\n", \u03ba, _u03ba);
  return \u03ba == _u03ba;
}

```

This little example program shows correct output with gcc:

> addresses 0x60103c and 0x601038
>
> values 1 and 0

and goes fundamentally wrong with icc:

> addresses 0x604a44 and 0x604a44
>
> values 0 and 0
