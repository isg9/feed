---
title: types and the web — wingolog
url: https://wingolog.org/archives/2011/01/11/types-and-the-web
published: "2011-01-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/01/11/types-and-the-web
---

# types and the web — wingolog

## [types and the web](/archives/2011/01/11/types-and-the-web)

11 January 2011 7:18 AM

- [guile](/tags/guile)
- [types](/tags/types)
- [web](/tags/web)
- [sxml](/tags/sxml)

*An essay expanding on the theme of types and the web. I wrote this for Guile's manual, but it applies generally, I think. The point about SXML can apply fruitfully to [python](http://git.gnome.org/browse/jhbuild/tree/jhbuild/utils/sxml.py) as well. -- Andy*

❧

It is a truth universally acknowledged, that a program with good use of data types, will be free from many common bugs. Unfortunately, the common practice in web programming seems to ignore this maxim. This subsection makes the case for expressive data types in web programming.

By "expressive data types", I mean that the data types *say* something about how a program solves a problem. For example, if we choose to represent dates using SRFI 19 date records (see [SRFI-19](http://www.gnu.org/software/guile/docs/master/guile.html/SRFI_002d19.html)), this indicates that there is a part of the program that will always have valid dates. Error handling for a number of basic cases, like invalid dates, occurs on the boundary in which we produce a SRFI 19 date record from other types, like strings.

With regards to the web, data types are help in the two broad phases of HTTP messages: parsing and generation.

Consider a server, which has to parse a request, and produce a response. Guile will parse the request into an HTTP request object (see [Requests](http://www.gnu.org/software/guile/docs/master/guile.html/Requests.html)), with each header parsed into an appropriate Scheme data type. This transition from an incoming stream of characters to typed data is a state change in a program---the strings might parse, or they might not, and something has to happen if they do not. (Guile throws an error in this case.) But after you have the parsed request, "client" code (code built on top of the Guile HTTP stack) will not have to check for syntactic validity. The types already make this information manifest.

This state change on the parsing boundary makes programs more robust, as they themselves are freed from the need to do a number of common error checks, and they can use normal Scheme procedures to handle a request instead of ad-hoc string parsers.

The need for types on the response generation side (in a server) is more subtle, though not less important. Consider the example of a POST handler, which prints out the text that a user submits from a form. Such a handler might include a procedure like this:

```
;; First, a helper procedure
(define (para . contents)
  (string-append "" (string-concatenate contents) ""))

;; Now the meat of our simple web application
(define (you-said text)
  (para "You said: " text))

(display (you-said "Hi!"))
-| You said: Hi!

```

This is a perfectly valid implementation, provided that the incoming text does not contain the special HTML characters `<`, `>`, or `&`. But this provision is not reflected anywhere in the program itself: we must *assume* that the programmer understands this, and performs the check elsewhere.

Unfortunately, the short history of the practice of programming does not bear out this assumption. A cross-site scripting (XSS) vulnerability is just such a common error in which unfiltered user input is allowed into the output. A user could submit a crafted comment to your web site which results in visitors running malicious Javascript, within the security context of your domain:

```
(display (you-said "
```
