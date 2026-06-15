---
title: Emacs Web Servlets
url: https://nullprogram.com/blog/2009/11/03/
published: "2009-11-03T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/11/03/
---

# Emacs Web Servlets

## [Emacs Web Servlets](/blog/2009/11/03/)

November 03, 2009

nullprogram.com/blog/2009/11/03/

Remember that [Emacs web server I wrote](/blog/2009/05/17/) back in May? Well, I got an e-mail last night from Chunye Wang containing a patch with a variant of my dynamic lisp idea, called "servlets" (not to be confused with Java servlets). Chunye had similar concept for an Emacs web server for a long time, but never implemented because Emacs lacked network functionality until recently (Specifically, [`make-network-process`](http://www.gnu.org/software/emacs/NEWS.22.1) in Emacs 22.1, June 2007). This led Chunye to find my implementation.

Again, you can clone/view the code here. I turned the patch into a series of commits,

```
git clone git://github.com/skeeto/emacs-http-server.git

```

This is some cool stuff here.

The servlets are simply functions installed under an " `httpd/`" namespace, where the trailing slash represents the server root. So, the function `httpd/example-servlet` will be executed when "/example-servlet" is requested from the server. The servlet runs on a temporary buffer, whose contents are served when the servlet function returns.

To assist in HTML generation, Chunye also wrote a function to turn an [S-expression](http://en.wikipedia.org/wiki/S-expression) into HTML, similar to the one I described in the web server previous post. Symbols are converted into strings, alists are attributes, and the `elisp` symbol indicates code to be executed, and the results used to generate HTML. For a simple hello word,

```cl
(html (head (title "hello world")) (body "hello world"))
```

And for some dynamic content, a die roller,

```cl
(defun httpd/roll-die (uri-query req uri-path)
  "Rolls a die with the requested number of sides (default 6)."
  (let ((sides
         (1- (string-to-number (or (cadr (assoc "sides" uri-query)) "6")))))
    (httpd-generate-html
     '(html
       (head
        (title "Die Roll Servlet"))
       (body
        (h1 "Die Roll Servlet")
        "You rolled a "
        (b
         (elisp (list (number-to-string (1+ (random sides)))))))))))
```

That one would be accessed from the browser with with " `/roll-die`" or " `/roll-die?sides=100`".

Chunye provided some sample servlets that list the buffers, with links that serve them up. There is also another servlet that will switch the current buffer, which I find compelling. All of Emacs' functionality is available to the servlet.

Now, to write a servlet that runs the Emacs psychiatrist ...

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs%20Web%20Servlets) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs+Web+Servlets).

« [Your BitTorrent Client is Probably Defective by Design](/blog/2009/10/26/)

» [Tweaking Emacs for Ant and Java](/blog/2009/12/06/)
