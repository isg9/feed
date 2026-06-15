---
title: 'A disimprovement observed from the outside: right angle&nbsp;brackets'
url: https://gustedt.wordpress.com/2013/12/15/a-disimprovement-observed-from-the-outside-right-angle-brackets/
published: "2013-12-15T23:16:09Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=2043
---

# A disimprovement observed from the outside: right angle&nbsp;brackets

It is long time that I didn’t look into C++, I have to admit. By coincidence I recently unearthed a hilarious example that I had once written that shows the difficulty of parsing some C++ code, as well as for compilers as for us poor humans. It all starts with the `>>` operator that (supposedly until C++11) could cause problems as in the following:

```
toto< tutu< 3 >> A;

```

Here the >> is (was) interpreted as \`right shift’ operator and thus this code would create a compile time error. C++11 changed this by introducing the possibility that in that case the right-shift-operator-token closes the two template angle brackets. The argument is that shift operators in template arguments are rare (which is probably true) and so this sacrifices some valid uses of that operator for the sake of causing less brain damage to C++ newbies.

Let’s look a bit deeper into the problem. Such a grammatical ambiguity would not be fun if we would not be able to produce valid but quite ambiguous code for pre-C++11:

```
template< unsigned len > unsigned int fun(unsigned int x);
typedef unsigned int (*fun_t)(unsigned int);
template< fun_t f > unsigned int fon(unsigned int x);

void total(void) {
   unsigned int A = fon< fun< 9 > >(1) >>(2);
   unsigned int B = fon< fun< 9 >>(1) > >(2);
}

```

Here `A` and `B` do not have much in common. For `A` we take the function `fon` that depends on function pointer `fun<9>` and call that function with argument `1`. The result of that call is then shifted by `2` to the right.

In contrast, for `B` we call the function `fon` that depends on function pointer `fun<4>` and pass it the argument `2`. (The 4 is 9 right-shifted by 1.)

So just because of some blanks that are spread differently the result can be completely different.

C++11 “fixed” such type of problems. Now this code isn’t valid anymore, which is probably a good thing as such.

But perhaps the reason is not what everyone would expect. The line

```
   unsigned int B = fon< fun< 9 >>(1) > >(2);

```

has a syntax error that only appears at the last > token. After the grammar change of C++11 the first part `fon< fun< 9 >>(1)` is now still valid, but interpreted differently than before. An error only occurs because now at the end there are two clashing > tokens.

For me, this really makes things worth than before. Instead of a clear tokenization that was compatible with C and the C preprocessor, C++ now has some sort of context dependent mixture (what is syntax, what is semantic, here). Hopefully there is no code that is valid in both variants of C++ and has different meaning.

At the origin of all of this difficulties lies a design flaw that dates back to the very start of templates, namely of using < and > tokens as angle brackets, which they aren’t. Nowadays, Unicode and html have the distinction between < and > on one hand and 〈 and 〉 on the other. The first two are comparison operators, the second two are opening and closing punctuation, respectively. Unfortunately, these fine distinctions for different characters as Unicode provides them had not been available at the time C++ emerged, and even less have they been available on standard terminals or encodings of that time.

Using symbols as replacement for different class symbols (namely brackets) and thereby making it incompatible with the basic tokenization is just a bad idea. Patching it with some extra rules makes things worth, and makes it, at the end, even more difficult to explain.

**Edit:** [the story continues with a really bad example, here](http://gustedt.wordpress.com/2013/12/18/right-angle-brackets-shifting-semantics/ "right angle brackets: shifting semantics")
