---
title: on the new posix — wingolog
url: https://wingolog.org/archives/2010/12/17/on-the-new-posix
published: "2010-12-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/12/17/on-the-new-posix
---

# on the new posix — wingolog

## [on the new posix](/archives/2010/12/17/on-the-new-posix)

17 December 2010 4:34 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [web](/tags/web)
- [http](/tags/http)
- [sxml](/tags/sxml)

*Here's some stuff I've been preparing for Guile's 1.9.14 release, which should come out later today. Readers interested in further web hackery should check out [Tekuti](http://gitorious.org/tekuti) for a larger example. Happy hacking!*

❧

When [Guile](http://www.gnu.org/software/guile/) started back in the mid-nineties, the GNU system was still focused on producing a good POSIX implementation. This is why Guile's [POSIX](http://www.gnu.org/software/guile/docs/master/guile.html/POSIX.html) support is good, and has been so for a while.

But times change, and in a way these days the web is the new POSIX: a standard and a motley set of implementations on which much computing is done. So today's Guile also supports the web at the programming language level, by defining common data types and operations for the technologies underpinning the web: [URIs](http://www.gnu.org/software/guile/docs/master/guile.html/URIs.html), [HTTP](http://www.gnu.org/software/guile/docs/master/guile.html/HTTP.html), and [XML](http://ssax.sourceforge.net/).

It is particularly important to define native web data types. Though the web is text in motion, programming the web in text is like programming with `goto`: muddy, and error-prone. Most current security problems on the web are due to treating the web as text instead of as instances of the proper data types.

In addition, common web data types help programmers to share code.

Well. That's all very nice and opinionated and such, but how do I use the thing? Read on!

**Hello, World!**

The first program we have to write, of course, is "Hello, World!". This means that we have to implement a web handler that does what we want. A handler is a function of two arguments and two return values:

```
(define (handler request request-body)
  (values response response-body))

```

In this first example, we take advantage of a short-cut, returning an alist of headers instead of a proper response object. The response body is our payload:

```
(define (hello-world-handler request request-body)
  (values '((content-type . ("text/plain")))
          "Hello World!"))

```

Now let's test it. Load up the web server module if you haven't yet done so, and run a server with this handler:

```
(use-modules (web server))
(run-server hello-world-handler)

```

By default, the web server listens for requests on `localhost:8080`. Visit that address in your web browser to test. If you see the string, `Hello World!`, sweet!

**Inspecting the Request**

The Hello World program above is a general greeter, responding to all URIs. To make a more exclusive greeter, we need to inspect the request object, and conditionally produce different results. So let's load up the request, response, and URI modules, and do just that.

```
(use-modules (web server)) ; you probably did this already
(use-modules (web request)
             (web response)
             (web uri))

(define (request-path-components request)
  (split-and-decode-uri-path (uri-path (request-uri request))))

(define (hello-hacker-handler request body)
  (if (equal? (request-path-components request)
              '("hacker"))
      (values '((content-type . ("text/plain")))
              "Hello hacker!")
      (not-found request)))

(run-server hello-hacker-handler)

```

Here we see that we have defined a helper to return the components of the URI path as a list of strings, and used that to check for a request to `/hacker/`. Then the success case is just as before -- visit `http://localhost:8080/hacker/` in your browser to check.

You should always match against URI path components as decoded by `split-and-decode-uri-path`. The above example will work for `/hacker/`, `//hacker///`, and `/h%61ck%65r`.

But we forgot to define `not-found`! If you are pasting these examples into a REPL, accessing any other URI in your web browser will drop your Guile console into the debugger:

```
:38:7: In procedure module-lookup:
:38:7: Unbound variable: not-found

Entering a new prompt.  Type `,bt' for a backtrace or `,q' to continue.
scheme@(guile-user) [1]>

```

So let's define the function, right there in the debugger. As you probably know, we'll want to return a 404 response.

```
;; Paste this in your REPL
(define (not-found request)
  (values (build-response #:code 404)
          (string-append "Resource not found: "
                         (unparse-uri (request-uri request)))))

;; Now paste this to let the web server keep going:
,continue

```

Now if you access `http://localhost/foo/`, you get this error message. (Note that some popular web browsers won't show server-generated 404 messages, showing their own instead, unless the 404 message body is long enough.)

**Higher-Level Interfaces**

The web handler interface is a common baseline that all kinds of Guile web applications can use. You will usually want to build something on top of it, however, especially when producing HTML. Here is a simple example that builds up HTML output using [SXML](http://en.wikipedia.org/wiki/SXML).

First, load up the modules:

```
(use-modules (web server)
             (web request)
             (web response)
             (sxml simple))

```

Now we define a simple templating function that takes a list of HTML body elements, as SXML, and puts them in our super template:

```
(define (templatize title body)
  `(html (head (title ,title))
         (body ,@body)))

```

For example, the simplest Hello HTML can be produced like this:

```
(sxml->xml (templatize "Hello!" '((b "Hi!"))))
=| Hello!Hi!

```

Much better to work with Scheme data types than to work with HTML as strings. Now we define a little response helper:

```
(define* (respond #:optional body #:key
                  (status 200)
                  (title "Hello hello!")
                  (doctype "\n")
                  (content-type-params '(("charset" . "utf-8")))
                  (content-type "text/html")
                  (extra-headers '())
                  (sxml (and body (templatize title body))))
  (values (build-response
           #:code status
           #:headers `((content-type
                        . (,content-type ,@content-type-params))
                       ,@extra-headers))
          (lambda (port)
            (if sxml
                (begin
                  (if doctype (display doctype port))
                  (sxml->xml sxml port))))))

```

Here we see the power of keyword arguments with default initializers. By the time the arguments are fully parsed, the `sxml` local variable will hold the templated SXML, ready for sending out to the client.

Instead of returning the body as a string, here we give a procedure, which will be called by the web server to write out the response to the client.

Now, a simple example using this responder, which lays out the incoming headers in an HTML table.

```
(define (debug-page request body)
  (respond
   `((h1 "hello world!")
     (table
      (tr (th "header") (th "value"))
      ,@(map (lambda (pair)
               `(tr (td (tt ,(with-output-to-string
                               (lambda () (display (car pair))))))
                    (td (tt ,(with-output-to-string
                               (lambda ()
                                 (write (cdr pair))))))))
             (request-headers request))))))

(run-server debug-page)

```

Now if you visit any local address in your web browser, we actually see some HTML, finally.

**Conclusion**

Well, this is about as far as Guile's built-in web support goes, for now. There are many ways to make a web application, but hopefully by standardizing the most fundamental data types, users will be able to choose the approach that suits them best, while also being able to switch between implementations of the server. This is a relatively new part of Guile, so if you have feedback, let us know, and we can take it into account. Happy hacking on the web!

## related articles

- [meta data](/archives/2010/12/13/meta-data)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [an in-depth look at the performance of guile's web server](/archives/2012/03/08/an-in-depth-look-at-the-performance-of-guiles-web-server)
- [fscons 2011: free software, free society](/archives/2011/11/28/fscons-2011-free-software-free-society)
- [types and the web](/archives/2011/01/11/types-and-the-web)

### 9 responses

1. Ben says:[17 December 2010 6:50 PM](#45f169be71124cb0c4aed4bb76400949f338f9a7)

   I don't know Guile, so forgive me for asking, but why all the parentheses (particularly closing parentheses)? Even with my most parentheses ridden JavaScript or Python code, I've never seen so many. o.o

   Even with a text editor that highlights matching pairs of paren's/brackets with good accuracy, I would think it'd be difficult to keep track with that many.

2. Gabriel says:[17 December 2010 8:17 PM](#751f5975341a32c03b2d426f2b3c5bd186fc3722)

   @Ben

   That's because Guile is a Scheme implementation, and thus a dialect of Lisp. The parentheses are there mainly to ease the manipulation of the program code, so that you can write new syntax on top of Scheme using its macro system.

   You don't need much care to close them correctly, you can just hammer the ')' key and auto-indent to see if you messed up. Also, functions are your friend.

   Give it a whirl, standard Scheme is a tiny language with few but interesting constructs. The syntax takes nothing to learn and is often used to make toy interpreters.

3. [wingo](http://wingolog.org/) says:[17 December 2010 8:49 PM](#0ecbf2f8a9ee2ec5ade2255e26539a0188b79efd)

   Hi Ben,

   I agree with everything Gabriel said, and would only like to add that I rarely type a closing parenthesis, because when I type '(', my editor inserts the ')' for me. But only when I hack in Guile, unfortunately; I often find myself wanting it for C, where you have almost as many balanced delimiters.

   See [Paredit](http://www.emacswiki.org/emacs/ParEdit) and [Using Guile in Emacs](http://www.gnu.org/software/guile/docs/master/guile.html/Using-Guile-in-Emacs.html#Using-Guile-in-Emacs), for more info.

4. Billy O'Connor says:[17 December 2010 10:11 PM](#276fde918b74f0308c1d4fa6252bc36a05533db3)

   Just gorgeous. I just checked out 1.9 from git. You made my xmas. :))))))))))))))))))

5. Ben says:[17 December 2010 10:52 PM](#06ca1cbef3a755eb9991ed58c185848a29e7e726)

   Thanks Gabriel, wingo. I might give it a look, as long as it doesn't have too many large dependencies. ;-)

6. [Ralph Moritz](http://makeyourselfobsolete.blogspot.com) says:[17 December 2010 11:13 PM](#78c202dfa1788c78b97d661bec732f12c196361d)

   Cool! Does Guile come with a Parenscript-like library for translating Scheme to JavaScript?

7. [kanru](http://kanru.info/blog) says:[18 December 2010 4:19 AM](#a3549ac46da027117da21db56afad805f1255f49)

   Great! And with Guile 1.9 you even can run server side ECMAScript within Guile itself ;-)

8. [Rudolf Olah](http://goaugust.com/) says:[18 December 2010 8:50 AM](#3328efb21dba4298646a16ba07ae4024f780ac70)

   This is awesome and all, but what I would like to see is something like Rosetta code. There are far too many tutorials out there where the explanations are identical but the code is different.

9. [dim](http://tapoueh.org) says:[20 December 2010 9:17 AM](#a8b6c3cbb23f41b54ffefd59831ae099acbdc283)

   Hi, congrats for your guile webserver!

   Now, what I think we need is not just another dynamic html rendering engine but a full application server that happens to be able to answer to http requests. The difference being in the life cycle of objects, you don't want to only have objects that live only enough to answer a specific request, any application needs some more.

   So you would end-up coding a full daemon application in guile/scheme and have some http handler objects in the mix, and they can talk to objects living in the daemon. Yaws (Erlang) supports that, for example.

   Regards,

Comments are closed.
