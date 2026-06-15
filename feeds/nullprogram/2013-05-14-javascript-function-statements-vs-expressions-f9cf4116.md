---
title: JavaScript Function Statements vs. Expressions
url: https://nullprogram.com/blog/2013/05/14/
published: "2013-05-14T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/05/14/
---

# JavaScript Function Statements vs. Expressions

## [JavaScript Function Statements vs. Expressions](/blog/2013/05/14/)

May 14, 2013

nullprogram.com/blog/2013/05/14/

The JavaScript `function` keyword has two meanings depending on how it’s used: as a *statement* or as an *expression*. It’s a statement when the keyword appears at the top-level of a block. This is known as a *function declaration*.

```
function foo() {
    // ...
}

```

This statement means declare a variable called `foo` in the current scope, create a closure named `foo`, and assign this closure to this variable. Also, this assignment is “lifted” such that it happens before any part of the body of the surrounding function is evaluated, including before any variable assignments.

Notice that the closure’s name is separate from the variable name. Except for a certain well-known JavaScript engine, closure/function objects have a read-only `name` property.

```
foo.name; // => "foo"

```

A name is required for function declarations, otherwise they would be no-ops. This name also appears in debugging backtraces.

A function’s name has different semantics in *function expressions*. The `function` keyword is an expression when used in an expression position of a statement.

```
var foo = function() {
    // ...
}

```

The function expression above evaluates to an anonymous closure, which is then assigned to the variable `foo`. This is *nearly* identical to the previous function declaration except for two details.

- Explicit variable assignments are never lifted, unlike function declarations. This assignment will happen exactly where it appears in the code.

- The resulting closure is anonymous. The `name` property, if available, will be an empty string. Furthermore, **the lack of name** **affects the scope of the function**. I’ll get back to that point in a moment.

### IIFEs

An immediately-invoked function expression (IIFE), used to establish a one-off local scope, is typically wrapped in parenthesis. The purpose of the parenthesis is to put `function` in an expression position so that it is a function expression rather than a function declaration.

```
(function() {
    // ... declare variables, etc.
}());

```

Another way to put `function` in an expression position is to precede it with an unary operator. This is an example of being clever instead of practical.

```
!function() {
    // ... declare variables, etc.
}();

```

If `function` is already in an expression position, the wrapping parenthesis are unnecessary. For example,

```
var foo = function() { return "bar"; }();
foo; // => "bar"

```

However, it may still be a good idea to wrap the IIFE in parenthesis just to help other programmers read your code. A casual glance that doesn’t notice the function invocation would assume a function is being assigned to `foo`. Wrapping a function expression with parenthesis is a well-known idiom for IIFEs.

### Function Name and Scope

What happens when a function expression is given a name? Two things.

1. The name will appear in the `name` property of the closure (if available). Also, the name will also show up in backtraces. This makes naming closures a handy debugging technique.

2. **The name becomes a variable in the scope of the function**. This means it’s possible to write recursive function expressions!

```
function maths() {
    return {
        // ...
        fact: function fact(n) {
            return n === 0 ? 1 : n * fact(n - 1);
        }
    };
}

maths().fact(10); // => 3628800

```

The `fact` function is evaluated as a function expression as part of this object literal. The variable `fact` is established in the scope of the function `fact`, assigned to the function itself, allowing the function to call itself. It’s a self-contained recursive function.

### Pop Quiz: Function Name and Scope

Given this, try to determine the answer to this problem in your head. What does the second invocation of `foo` evaluate to?

```
function foo() {
    foo = function() {
        return "function two";
    };
    return "function one";
}

foo(); // => "function one"
foo(); // => ???

```

Here’s where we come to the major difference between function declarations and function expressions. The answer is `"function two"`. Even though functions declarations create named functions, **these** **functions do not have the implicit self-named variable in its scope**. Unless this variable is declared explicitly, the name will refer to a variable in a containing scope.

This has the useful property that a function can re-define itself *and* be correctly named at the same time. If the function needs to perform expensive first-time initialization, such reassignment can be used to do it lazily without exposing any state *and* without requiring an is-initialized check on each invocation. For example, this trick is exactly how Emacs autoloading works.

If this function declaration is converted to what *appears* to be the equivalent function expression form the difference is obvious.

```
var foo = function foo() {
    foo = function() {
        return "function two";
    };
    return "function one";
};

foo(); // => "function one"
foo(); // => "function one"

```

The reassignment happens in the function’s scope, leaving the outer scope’s assignment intact. For better or worse, even ignoring assignment lifting, there’s no way to perfectly emulate function declaration using a function expression.

- [javascript](/tags/javascript/)
- [lang](/tags/lang/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20JavaScript%20Function%20Statements%20vs.%20Expressions) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=JavaScript+Function+Statements+vs.+Expressions).

« [Inventing a Datetime Web Service](/blog/2013/05/11/)

» [Load Libraries in Skewer with Bower](/blog/2013/05/18/)
