---
title: javascript eval considered crazy — wingolog
url: https://wingolog.org/archives/2012/01/12/javascript-eval-considered-crazy
published: "2012-01-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/01/12/javascript-eval-considered-crazy
---

# javascript eval considered crazy — wingolog

## [javascript eval considered crazy](/archives/2012/01/12/javascript-eval-considered-crazy)

12 January 2012 4:34 PM

- [javascript](/tags/javascript)
- [javascriptcore](/tags/javascriptcore)
- [ecmascript](/tags/ecmascript)
- [igalia](/tags/igalia)

Peoples. I was hacking recently on JavaScriptCore, and I came to a realization: JavaScript's `eval` is absolutely crazy.

I mean, I thought I knew this before. But... words fail me, so I'll have to show a few examples.

**`eval` and introduced bindings**

This probably isn't worth mentioning, as you probably know it, but `eval` can introduce lexical bindings:

```
 > var foo = 10;
 > (function (){ eval('var foo=20;'); return foo; })()
 20
 > foo
 10

```

I find this to be pretty insane already, but I knew about it. You would think though that `var x = 10;` and `eval('var x = 10;');` would be the same, though, but they're not:

```
 > (function (){ var x = 10; return delete x; })()
 false
 > (function (){ eval('var x = 10;'); return delete x; })()
 true

```

`eval`-introduced bindings do not have the `DontDelete` property set on them, according to the post-hoc language semantics, so unlike proper lexical variables, they may be deleted.

**when is `eval` really `eval`?**

Imagine you are trying to analyze some JavaScript code. If you see `eval(...)`, is it really `eval`?

Not necessarily.

`eval` pretends to be a regular, mutable binding, so it can be rebound:

```
 > eval = print
 > eval('foo')
 foo // printed

```

or, shadowed lexically:

```
 > function () { var eval = print; eval('foo'); }
 foo // printed

```

or, shadowed dynamically:

```
 > with({'eval': print}) { eval('foo'); }
 foo // printed

```

You would think that if you can somehow freeze the `eval` binding on the global object, and verify that there are no `with` forms, and no statements of the form `var eval = ...`, that you could guarantee that `eval` is `eval`, but that is not the case:

```
 > Object.freeze(this);
 > (function (x){ return [eval(x), eval(x)]; })('var eval = print; 10')
 var eval = print; 10 // printed, only once!
 10,

```

(Here the first `eval` created a local binding for `eval`, and evaluated to 10. The second `eval` was actually a `print`.)

Crazy!!!!

**an eval by any other name**

So `eval` is an identifier that can be bound to another value. OK. One would expect to be able to bind another identifier to `eval`, then. Does that work? It seems to work:

```
 > var foo = eval;
 > foo('foo') === eval;
 true

```

But not really:

```
 > (function (){ var quux = 10; return foo('quux'); } )()
 Exception: ReferenceError: Can't find variable: quux

```

`eval` by any other name isn't `eval`. (More specifically, `eval` by any other name doesn't have access to lexical scope.)

Note, however, the mere presence of a shadowed declaration of `eval` doesn't mean that `eval` *isn't* `eval`:

```
 > var foo = 10
 > (function(x){ var eval = x; var foo = 20; return [x('foo'), eval('foo')] })(eval)
 10,20

```

Crazy!!!!

**strict mode restrictions**

ECMAScript 5 introduces "strict mode", which prevents `eval` from being rebound:

```
 > (function(){ "use strict"; var eval = print; })
 Exception: SyntaxError: Cannot declare a variable named 'eval' in strict mode.
 > (function(){ "use strict"; eval = print; })
 Exception: SyntaxError: 'eval' cannot be modified in strict mode
 > (function(){ "use strict"; eval('eval = print;'); })()
 Exception: SyntaxError: 'eval' cannot be modified in strict mode
 > (function(x){"use strict"; x.eval = print; return eval('eval');})(this)
 Exception: TypeError: Attempted to assign to readonly property.

```

But, since strict mode is embedded in "classic mode", it's perfectly fine to mutate `eval` from outside strict mode, and strict mode has to follow suit:

```
 > eval = print;
 > (function(){"use strict"; return eval('eval');})()
 eval // printed

```

The same is true of non-strict lexical bindings for `eval`:

```
 > (function(){ var eval = print; (function(){"use strict"; return eval('eval');})();})();
 eval // printed
 > with({'eval':print}) { (function(){ "use strict"; return eval('eval');})() }
 eval // printed

```

An engine still has to check at run-time that `eval` is really `eval`. This crazy behavior will be with us as long as classic mode, which is to say, forever.

Strict-mode eval does have the one saving grace that it cannot introduce lexical bindings, so implementors do get a break there, but it's a poor consolation prize.

**in summary**

What can an engine do when it sees `eval`?

Not much. It can't even prove that it is actually `eval` unless `eval` is not bound lexically, there is no `with`, there is no intervening non-strict call to any identifier `eval` (regardless of whether it is `eval` or not), and the global object's `eval` property is bound to the blessed eval function, *and* is configured as `DontDelete` and `ReadOnly` (not the default in web browsers).

But the very fact that an engine sees a call to an identifier `eval` poisons optimization: because `eval` can introduce variables, the scope of free variables is no longer lexically apparent, in many cases.

I'll say it again: crazy!!!

## related articles

- [on generators](/archives/2013/02/25/on-generators)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [ffconf 2014](/archives/2014/11/09/ffconf-2014)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)

### 13 responses

1. vitriolix says:[12 January 2012 5:19 PM](#cc59be3ce24a78b9df0331f2c5d4b7ab40554816)

   Nice post. This belongs on wtfjs

2. [Jeff Walden](http://whereswalden.com/) says:[12 January 2012 11:24 PM](#9b19df3823c77fc9437b0910849f2b084405a971)

   So much truth here.

   I took a slightly different tack in my own argument for people to not use `eval`, in that I specifically focused on the optimizations it disabled. This was in the context of a post discussing implementation of proper strict mode variable binding in `eval`'d code. Nonetheless, we are in agreement that `eval` is insane, and no one should use it.

   http://whereswalden.com/2011/01/10/new-es5-strict-mode-support-new-vars-created-by-strict-mode-eval-code-are-local-to-that-code-only/

3. [Jeff Walden](http://whereswalden.com/) says:[12 January 2012 11:24 PM](#5ccc70a001463f45b61b08b8ec2301ad13af7d82)

   Hmm, didn't linkify that, let's try again:

   [http://whereswalden.com/2011/01/10/new-es5-strict-mode-support-new-vars-created-by-strict-mode-eval-code-are-local-to-that-code-only/](http://whereswalden.com/2011/01/10/new-es5-strict-mode-support-new-vars-created-by-strict-mode-eval-code-are-local-to-that-code-only/)

4. Benjamin Otte says:[13 January 2012 1:40 AM](#6901dfedcdc0c67fbf2e7842f6358f29a24d1578)

   This post is the perfect example of why I never know if I should love or hate the browser guys - on the one hand they invent something like eval(), on the other hand they make extra sure that links in their blog comments are properly linkified. ;)

5. [John-David Dalton](http://allyoucanleet.com) says:[13 January 2012 8:23 AM](#81d219034d207a46e9e418090c60c9e3b03c1b8b)

   The behavior you're seeing when you assign \`eval\` to a variable and execute it is defined by ES5. Indirect calls to \`eval\` like \`var a = eval; a(code)\` or \`(0, eval)(code)\` or \`window.eval(code)\` are treated as a global evals.

   http://es5.github.com/#x10.4.2

   Juriy Zaytsev (@kangax) has a post on global eval:

   http://perfectionkills.com/global-eval-what-are-the-options/

   and another post examining the \`delete\` operator and \`eval\`:

   http://perfectionkills.com/understanding-delete/#deleting\_variables\_via\_eval

6. Zachary Kessin says:[13 January 2012 1:45 PM](#22a61c5dbee7ba7a8303c3ab9529a7bd78aca278)

   This is why most of us say "Eval" really just "Evil" with a spelling mistake. with is also somewhat problematic but that is a different conversation.

   I found if you follow the advice in JS:TGP you will do much better with Javascript

7. Michael Trotter says:[13 January 2012 5:43 PM](#104c18d44d433a640772b04ba499400dd3caff6d)

   Very interesting..

   Although, I just tried this snippet from about (in the node.js interpreter):

   \> var foo = 10

   \> (function(x){ var eval = x; var foo = 20; return \[x('foo'), eval('foo')\] })(eval)

   It printed 10, 10

8. [Rauol Iglesias](http://hyperboleandahalf.blogspot.com/) says:[15 January 2012 2:17 PM](#8a1a8a88319adbb6d615637e3f8f0f814ac644f6)

   there is a new post on hyperbole and a half

9. matt dales says:[16 January 2012 10:45 AM](#4434aa7b541f2b95737b2eb7f076f7eeed407f1e)

   your first example seems flawed.

   var foo = 10; --declares a variable 'foo' in the global scope.

   (function (){

   eval('var foo=20;');

   --using 'var' here declares 'foo' in the local function's scope.

   return foo;

   })()

   foo

   in that example the global var remains 10 and the local version is set to 20 returned and deleted as per normal javascript rules.

10. Brandon says:[17 January 2012 6:35 PM](#ec3fa0a2a6c6779ac1f8988377b58a360c93ee4d)

    I'm inclined to agree with Zachary: there's no good reason to use eval. Quite apart from the performance impacts, using it can potentially expose your users to malicious code. Thankfully, browsers have given us there is now another way to parse JSON, but there are still plenty of js "ninjas" out there who think that cleaver uses of eval are a means of demsonstrating their coding prowess.

    Andy, my question to you as a scheme programmer, and language implementer, is whether you think there is a good case for including eval or something similar into the language.

11. Brandon says:[17 January 2012 6:36 PM](#c4df5c994784c390fb980424446cf23dfc564875)

    On a side note, have you considered implementing a way to edit comments post facto?

12. Erik says:[10 July 2013 6:19 AM](#053a5bbc7878c533cc21410334cf51af632b8ee5)

    Code generation is one reason to use eval, at least when your language doesn't have real code manipulation (e.g. macros).

    Introducing new variables in it's own external scope is crazy though. (who thought that was a good idea?)

13. [Jason](https://concertbox.wordpress.com/) says:[17 October 2016 11:37 AM](#41136c1e880e514f23c22b08846a40f3fd5b9a48)

    \> var foo = 10;

    \> (function (){ eval('var foo=20;'); return foo; })()

    20

    \> foo

    10

    it was some kind of a function?

Comments are closed.
