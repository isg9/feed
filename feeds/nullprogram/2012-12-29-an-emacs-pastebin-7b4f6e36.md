---
title: An Emacs Pastebin
url: https://nullprogram.com/blog/2012/12/29/
published: "2012-12-29T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/12/29/
---

# An Emacs Pastebin

## [An Emacs Pastebin](/blog/2012/12/29/)

December 29, 2012

nullprogram.com/blog/2012/12/29/

Luke is doing an interesting threefive-part tutorial on writing a pastebin in PHP: [PHP Like a Pro](http://terminally-incoherent.com/blog/2012/12/17/php-like-a-pro-part-1/) ( [2](http://terminally-incoherent.com/blog/2012/12/19/php-like-a-pro-part-2/), [3](http://terminally-incoherent.com/blog/2012/12/26/php-like-a-pro-part-3/), [4](http://terminally-incoherent.com/blog/2013/01/02/php-like-a-pro-part-4/), [5](http://terminally-incoherent.com/blog/2013/01/04/php-like-a-pro-part-5/)). The tutorial is largely an introduction to the set of tools a professional would use to accomplish a more involved project, the most interesting of which, for me, is [Vagrant](http://vagrantup.com/).

Because I have [no intention of ever using PHP](http://me.veekun.com/blog/2012/04/09/php-a-fractal-of-bad-design/), I decided to follow along in parallel with my own version. I used Emacs Lisp with my [simple-httpd](/blog/2012/08/20/) package for the server. I really like my servlet API so was a lot more fun than I expected it to be! Here’s the source code,

- [https://github.com/skeeto/emacs-pastebin](https://github.com/skeeto/emacs-pastebin)

Here’s what it looked like once I was all done,

[![](/img/screenshot/pastebin-thumb.png)](/img/screenshot/pastebin.png)

It has syntax highlighting, paste expiration, and light version control. The server side is as simple as possible, consisting of only three servlets,

- `/pastebin/`: static files
- `/pastebin/get`: serves (immutable) pastes in JSON
- `/pastebin/post`: accepts new pastes in JSON, returns the ID

A paste’s JSON is the raw paste content plus some metadata, including post date, expiration date, language (highlighting), parent paste ID, and title. That’s it! The server is just a database and static file host. It performs no dynamic page generation. Instead, the client-side JavaScript does all the work.

For you non-Emacs users, the repository has a `pastebin-standalone.el` which can be used to launch a standalone instance of the pastebin server, so long as you have Emacs on your computer. It will fetch any needed dependencies automatically. See the header comment of this file for instructions.

### IDs

A paste ID is four or more randomly-generated numbers, letters, dashes or underscores, with some minor restrictions ( `pastebin-id-valid-p`). It’s appended to the end of the servlet URL.

- `/pastebin/`
- `/pastebin/get/`

In the first case, the servlet entirely ignores the ID. Its job is only to serve static files. In the second case the server looks up the ID in the database and returns the paste JSON.

The client-side inspects the page’s URL to determine the ID currently being viewed, if any. It performs an asynchronous request to `/pastebin/get/` to fetch the paste and insert the result, if found, into the current page.

Form submission isn’t done the normal way. Instead, the submission is intercepted by an event handler, which wraps the form data up in JSON (much cleaner to parse!) and sends it asynchronously to `/pastebin/post` via POST. This servlet inserts the paste in the database and responds in `text/plain` with the paste ID it generated. The client-side then redirects the browser to the paste URL for that paste.

### Features

As I said, the server performs no page generation, so syntax highlighting is done in the client with [highlight.js](http://softwaremaniacs.org/soft/highlight/en/). I *could* have used [htmlize](http://emacswiki.org/emacs/Htmlize) and supported any language that Emacs supports. However, I wanted to keep the server as simple as possible, and, more importantly, I *really* don’t trust Emacs’ various modes to be secure in operating on arbitrary data. That’s a huge attack surface and these modes were written without security in mind (fairly reasonable). It’s actually a deliberate feature for Emacs to automatically `eval` Elisp in comments [under certain circumstances](http://www.gnu.org/software/emacs/manual/html_node/emacs/Specifying-File-Variables.html).

Version control is accomplished by keeping track of which paste was the parent of the paste being posted. When viewing a paste, the content is also placed in a textarea for editing. Submitting this form will create a new paste with the current paste as the parent. When viewing a paste that has a parent, a “diff” option is provided to view a diff patch of the current paste with its parent (see the screenshot above). Again, the server is dead simple, so this patch is computed by JavaScript after fetching the parent paste from the server.

### Databases

As part of my fun I made a generic database API for the servlets, then implemented three different database backends. I used eieio, Emacs Lisp’s CLOS-like object system, to implement this API. Creating a new database backend is just a matter of making a new class that implements two specific methods.

The first, and default, implementation uses an Elisp hash table for storage, which is lost when Emacs exits.

The second is a flat-file database. I estimate it should be able to support at least 16 million different pastes gracefully. The on-disk format for pastes is an s-expression. Basically, this is read by Emacs, expiration date checked, converted to JSON, then served to the client.

To my great surprise there is practically no support for programmatic access to a SQL database from *GNU* Emacs Lisp (other Emacsen do). The closest I found was [pg.el](http://www.online-marketwatch.com/pgel/pg.html), which is asynchronous by necessity. However, the specific target I had in mind was SQLite.

I *did* manage to implement a third backend that uses SQLite, but it’s a big hack. It invokes the `sqlite3` command line program once for every request, asking for a response in CSV — the only output format that seems to escape unambiguously. This response then has to be parsed, so long as it’s not too long to blow the regex stack.

*Update February 2014*: I have [found a solution to this problem](/blog/2014/02/06/)!

### Future

This has been an educational project for me. As a tutorial and for practice I’ll probably write the server again from scratch using other languages and platforms (Node.js and Hunchentoot maybe?), keeping the same front-end.

- [elisp](/tags/elisp/)
- [emacs](/tags/emacs/)
- [javascript](/tags/javascript/)
- [web](/tags/web/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20An%20Emacs%20Pastebin) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=An+Emacs+Pastebin).

« [How a simple-httpd Vulnerability Slipped In](/blog/2012/12/18/)

» [Clojure and Emacs for Lispers](/blog/2013/01/07/)
