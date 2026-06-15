---
title: JavaScript Strings as Arrays
url: https://nullprogram.com/blog/2012/11/15/
published: "2012-11-15T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/11/15/
---

# JavaScript Strings as Arrays

## [JavaScript Strings as Arrays](/blog/2012/11/15/)

November 15, 2012

nullprogram.com/blog/2012/11/15/

### Lisp

One thing I enjoy about Common Lisp is its general treatment of sequences. (In fact, I wish it went further with it!) Functions that don’t depend on list-specific features generally work with any kind of sequence. For example, `remove-duplicates` doesn’t just work with lists, it works on any sequence.

```
(remove-duplicates '(a b b c))  ; list
=> (A B C)

(remove-duplicates #(a b b c))  ; array
=> #(A B C)

```

Functions like `member` and `mapcar` require lists because their behavior explicitly uses them. The general sequence version of these are `find` and `map`. Writing a new sequence function means sticking to these generic sequence functions, particularly `elt` and `subseq` rather than the more specialized accessors.

A string is just a one-dimensional array — a vector — with elements of the type *character*. This means all sequence functions also work on strings.

```
(make-array 10 :element-type 'character :initial-element #\a)
=> "aaaaaaaaaa"

(remove-duplicates "abbc")
=> "abc"

(map 'string #'char-upcase "foo")
=> "FOO"

(reverse "foo")
=> "oof"

```

There is no special set of functions just for operating on strings (except those for string-specific operations). Strings are as powerful and flexible as any other sequence. This is very convenient.

### JavaScript

Unfortunately, JavaScript strings aren’t *quite* arrays. They look and act a little bit like arrays, but they’re missing a few of the useful methods.

```
var foo = "abcdef";

foo[1]
=> "b"

foo.length
=> 6

foo.reverse()  // error, no method 'reverse'

```

Notice that, when indexing, it returns a one-character string, not a single character. This is because there’s no character type in JavaScript. It would have been interesting if JavaScript had gone the Elisp route, where there’s no character type but instead characters are represented by integers, with some sort of character literal for using characters in code. This sort of thing can be emulated with the `charCodeAt()` method.

To work around the strings-are-not-arrays thing, strings can be converted to arrays with `split()`, manipulated as an array, and restored with `join()`.

```
foo.split('').reverse().join('')
=> "fedcba"

```

The string method `replace` can act as a stand-in for `map` and `filter`. The replacement argument can be a function, which will be called on each match. If a single character at a time is selected for replacement then what’s left is the `map` method.

```
// Map over each character
foo.replace(/./g, function(c) {
    return String.fromCharCode(c.charCodeAt(0) + 10);
});
=> "klmnop"

```

For `filter`, an empty string would be returned in the case of the predicate returning `false` and the original match in the case of `true`.

```
foo.replace(/./g, function(c) {
    if ("xyeczd".indexOf(c) >= 0)
        return c;
    else
        return '';
});
=> "cde"

```

In most cases, typical use of regular expressions would serve the need for the `filter()` method, so this is mostly unnecessary. For example, the above could also be done like so,

```
foo.replace(/[^xyeczd]/g, '');

```

Another way to fix the missing methods would be to simply implement the Array methods for strings and add them to the String prototype, but that’s generally considered bad practice.

- [javascript](/tags/javascript/)
- [lisp](/tags/lisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20JavaScript%20Strings%20as%20Arrays) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=JavaScript+Strings+as+Arrays).

« [JavaScript's Quirky eval](/blog/2012/11/14/)

» [JavaScript Debugging Challenge](/blog/2012/11/19/)
