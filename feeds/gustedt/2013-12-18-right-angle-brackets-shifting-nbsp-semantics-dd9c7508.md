---
title: 'right angle brackets: shifting&nbsp;semantics'
url: https://gustedt.wordpress.com/2013/12/18/right-angle-brackets-shifting-semantics/
published: "2013-12-18T23:29:29Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=2083
---

# right angle brackets: shifting&nbsp;semantics

As I showed int [this post](http://gustedt.wordpress.com/2113/12/15/a-disimprovement-observed-from-the-outside-right-angle-brackets/ "A disimprovement observed from the outside: right angle brackets"), using > as right angle brackets was not a particularly good idea, but trying to patch this misdesign even makes it worth. After a bit of experimenting I found an expression that is in fact valid for both, C++98 and `C++11`, but that has a different interpretation in both languages:

```
fon< fun< 1 >>::three >::two >::one

```

So if you have to maintain a large code base with templates that depend on integers that are perhaps produced automatically by some tools, be happy, you will not be out of work for a while: changing your compiler to `C++11` might change the semantics of your code.

Let’s have a look how this expression is parsed

```
fon< fun< 1 >>::three >::two >::one    // in C++98
          -----------
     -----------------------
-----------------------------------

```

```
fon< fun< 1 >>::three >::two >::one    // in C++11
     --------
---------------------
----------------------------
-----------------------------------

```

## C++98

So for C++98 the innermost level has

```
1 >> ::three

```

the shift operator applied to 1 with an operand of a global identifier `::three`, say X. This X is now template argument to `fun`

```
fun< X >::two

```

the member `two` of template class `fun`, lets call this Y. The last level is then

```
fon< Y >::one

```

The member `one` of template class `fon`.

## C++11

```
fun< 1 >

```

is the template class `fun` for parameter 1, say `fun1`.

```
fon< fun1 >::three

```

a member `three` of the fon class, say Z.

```
Z > ::two

```

Z compared to the global symbol `two`, resulting in a bool W.

```
W > ::one

```

W compared to a global symbol `one`.

## An instantiation of that mess

Now that we have seen that in both languages that expression can be parsed, we want to have an instantiation that is correct in both, but where the semantic differs. In fact the following is a correct program, that allows to distinguish C++98 and C++11 compilers.

```
#include <stdio.h>

bool const one = true;
int const two = 2;
int const three = 3;

template< int > struct fun {
  typedef int two;
};

template< class T > struct fon {
  static int const three = ::three;
  static bool const one = ::one;
};

bool const cpp98 = fon< fun< 1 >>::three >::two >::one;
//bool const as_cpp98 = fon< fun< (1 >>::three) >::two >::one; // valid for both
//bool const as_cpp11 = (fon< fun< 1 >>::three) >::two >::one; // not valid in C++98

int main(void) {
  printf("this is a %sC++11 compiler, version is %ld\n", (cpp98 ? "pre-" : ""), __cplusplus);
}

```

In the interpretation of both, the expression in question is of type `bool`, but for different reasons. For C++98 it is because the expression is the class member `one` of `fon`. This is always a `bool` with value `true`.

For C++11, it is the result of a comparison between two `bool` values, so `bool`, too. The value is `false`, here.

## Conclusion

Not only is C++ the victim of some poor choices that have been made at the start. The committee doesn’t succeed in avoiding such mistakes for new versions of the standard. Maintaining a C++ code base where the language shifts under your feet was and still *is* a nightmare. In my limited experience, before it was a question of syntax changes that errored out under compilation. Now we have a state where valid code for one version remains valid for the other, but the result is different.

That language is definitively not for me.
