---
title: slime — wingolog
url: https://wingolog.org/archives/2006/01/02/slime
published: "2006-01-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/01/02/slime
---

# slime — wingolog

## [slime](/archives/2006/01/02/slime)

2 January 2006 11:12 PM

- [scheme](/tags/scheme)
- [python](/tags/python)

I've been meaning to try out [SLIME](http://common-lisp.net/project/slime/) for a long time, and finally got around to it a couple of months ago. Let me say that I'm very, very impressed with the rich development environment Common Lisp people have made for themselves. The python environment I use at work (Emacs for editing, [ipython](http://ipython.scipy.org/) for experimenting, occasional usage of grep) pales in comparison.

I told myself then that I'd have to write down what it is that makes SLIME so great, but that got put aside in the runup to GStreamer 0.10. Now with a bit more time on my hands I've taken another look at it. It's still impressive.

#### Implementation notes

I have to touch on the implementation first, because otherwise the rest won't make sense. Most emacs and vi users are accustomed to specialized editor modes for different languages: one for python to help in indentation, one for C, etc. These modes rely on textual, syntactic analysis of the source code file to try to anticipate what you want the editor to do. However it's never perfect; local variables that have the same name as builtin functions in python still get highlighted as if they were builtins, some macros might upset the indenter in C, etc.

SLIME takes a different approach to "editor modes". It's a client-server architecture; the Emacs part of SLIME connects to an external Lisp process, which is running a SLIME server. The two sides communicate via a special RPC protocol.

If SLIME existed for Python, it would not highlight builtins based on their name. Instead it would have an inferior python interpreter running the whole time, and when the user types in `map`, it would ask the python interpreter what `map` is. Based on that answer it would then choose how to highlight the word, and maybe even tell the programmer what the arguments to `map` are.

The server is written in mostly-portable Common Lisp, so that it runs on something like 9 implementation of the language. Because the protocol is the same, SLIME can interact with all implementations in the same way. This is of great interest to CL programmers, but even Python has a couple of other slightly incompatible implementations, so the idea of communicating via sockets with an external server is still a good one.

I'm going to artificially divide up SLIME's feature set into a few categories: writing new code, changing existing code, finding code, and debugging. To make the examples more relevant for most people, I'll make some speculations throughout about what these features would mean if they were implemented for Python.

#### Writing code

I've amazed my co-worker Thomas, a vi user, with the emacs command `dabbrev-expand`, normally bound to `M-/`. For example typing `p r o g M-/` in this file expands `prog` out to `programmers`. However because this expansion is just textual, it often takes a few presses of the `M-/` before I get the word I want. For example, when looking for a method, a module name will expand out, or a word from comments.

SLIME extends emacs such that `M-TAB` tries to expand the symbol at point (the cursor) to other symbols available in that package (module) in Lisp. If I type `a p p M-TAB`, SLIME tells me that the two symbols `append` and `apply` are available for me to use, and if there is only one possible completion it goes ahead and finishes the word.

In addition "fuzzy completion" is available as an option, such that `norm-df` could expand to `least-positive-normalized-double-float`. Neat eh? In Python this would be more usefully implemented to autocomplete attributes as well.

Once you have the function you want, moving to the arguments by pressing the space bar ( `SPC`) will show the function's argument list in the minibuffer. For example typing `( a p p e n d SPC` in a Lisp file or at the REPL (listener) will cause `(append &rest lists)` to show up in the minibuffer. This describes the kind of arguments that `append` can take.

For more complicated functions with lots of keyword arguments, it's sometimes better to have the form expand out into your source itself. This can be done with `C-c C-s`, the binding for `slime-complete-form`. For example, if you have this in your source:

```
  (find 17

```

and you press `C-c C-s`, it will expand out to:

```
  (find 17 sequence :from-end from-end :test test
        :test-not test-not :start start :end end :key key)

```

If you need full documentation on `find`, you can put the point over `find` and run `slime-describe-function` via `C-c C-f`, and you get full documentation on the routine displayed in another pane. The same can be done with other variables via `slime-describe-symbol`.

Thankfully, unlike C, there is one indentation custom agreed upon by most all Lisp programmers. The convention depends on the first element of an expression. SLIME makes sure that the editor knows how to indent properly by checking the type of the operator with the Lisp environment. (This is a pet peeve of mine when hacking Guile, that Emacs often doesn't know how to indent my macros.)

#### Changing code

Of course, programming in a dynamic language usually doesn't follow a strict write-compile-run-debug cycle. Normally, people try to make small pieces that work, and then put them together. SLIME offers a number of conveniences that decrease the distance between writing code and seeing what it does.

Common Lisp is a dynamic, compiled language. You can recompile just one function, and have a running program take up that new definition. In fact, because SLIME has a connection to your running program, there are hotkeys to do just that. Like compilers for other languages, Lisp compilers can issue warnings corresponding to particular pieces of code. These warnings then show up in your source files as being underlined. For example, after compiling a function, unused variables will become underlined, and if you hover the mouse over the variables, the text of the compiler warning will show up as a tooltip.

In the hypothetical SLIME for Python, you would expect that the same would be possible -- send over a new method implementation, monkeypatch the class, and then all instances will use that method (even existing instances). Also if [pychecker](http://pychecker.sourceforge.net/) is available, it can be run on that function to give you similar kinds of warning information that can be translated to tooltips in the source files. All this is possible without having to restart your program.

(As a parenthesis, Common Lisp's object system was designed to allow reloading class definitions, and lazily migrating existing instances over to the new layout. I don't think Python can do this yet, but I'm not sure.)

#### Finding code

Oftentimes a programmer decides to change some aspect of a program's behavior, but is unfamiliar with the code concerned. This happens especially often on multi-programmer projects. The search for the proper code to change usually takes more time than the change itself. SLIME offers a number of facilities to speed up the search.

The traditional way of cross-referencing source code is via the use of [ctags](http://ctags.sourceforge.net/)/etags. First you set up some makefile rules for producing a TAGS file, which syntactically analyzes all of the source files in a project and notes the names and locations of all of the function definitions. Automake can assist with this. Then in your editor with the cursor on a function invocation, you press some key combination ( `M-.` in emacs) and your editor takes you to the definition.

This method is useful, but also a pain. If there are multiple functions with the same name (particularly bad in object-oriented languages), it takes a while to find the right one. Then there's the fact that you have to set up your source to build TAGS files, and keep them up-to-date. Finally, at least in Emacs, using TAGS files from multiple project is a PITA.

SLIME instead offers a meta-point facility (named after the key combination, `M-.`) that asks Lisp where the function was defined. SLIME saves where you were so you can go back there later. The saved location list is a stack, so you can meta-point out more than one time, and when you pop back via `M-,` you'll be in the last place you pressed `M-.`. This eliminates the need for syntactical analysis via TAGS files, and removes the duplicate-name problem.

If the Lisp implementation supports it, SLIME can offer much more detailed cross-referencing information: where a function is called, where a variable is referenced, where a macro is expanded, and where a generic function is specialized (akin to implementing a virtual method). Python could offer this information as well, but you would have to run a function to walk all modules, examining functions' bytecode and building an index.

Finally, SLIME offers easy access to all of the interactive documentation that Lisp knows about: docstrings on functions, documentation properties on variables, and the hyperspec, a heavily cross-referenced language reference. A SLIME for Python would offer similar facilities.

#### Debugging

I hardly ever use the debugger in Python. Part of it is that the code I work on mostly involves asynchronous calls, which are tough to debug, but mostly it's that I always forget how to use it. There are many occasions on which a debugger would be useful but I can't be bothered to remember how to start it.

SLIME makes all this very easy. Whenever a condition (like an exception) is signaled, the debugger is brought up in a transient pane (emacs' equivalent of a dialog box). From there you get a nice backtrace, a description of the error, and a list of [restarts](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html). The normal case is to press `q` and the last available restart will be chosen, which for example in the REPL will abort the evaluation.

Simple backtraces are readily had from ipython however. SLIME goes significantly beyond this. Each frame in the backtrace can be expanded out to show all of the local variables in the frame. This is done by putting the cursor on the frame and pressing enter.

If you press enter on a local variable, you start up the inspector, which opens in another transient pane. The inspector is really neat. It tells you everything it knows about a value. For example, the number 3 is described as an integer, then is shown in many different bases and as a time value. Compound values like closures are decomposed into their constituents. From a closure you can get the function object, together with the local variables that the function closes over. The function object itself can be inspected to show the source info (where it came from), the native code that the function compiled to, the arguments, etc. The inspector keeps a stack of objects so you can dig down and come back up to where you were. I would kill to have this in Python.

It would be really useful to be able to open up such a debugger if an exception has to be trapped, for example if an error occurs in a function run from the GObject main loop. As it is, I have to deal with python printing a backtrace, and I don't even get to know which values were on the stack.

Of course, SLIME also supports the traditional debugger interface of breakpoints and single-stepping. There are hotkeys assigned to break on certain lines, and also to trace/untrace functions. Common Lisp specifies a statistical profiling interface, so that also can be enabled and disabled from within the editor. Some Lisp implementations can give [profiling information for each instruction](http://groups.google.com/group/comp.lang.lisp/msg/ed551830b6ba8697) executed by the CPU.

#### Things I didn't know python-mode could do

Until writing this, I hadn't fully investigated python-mode. It doesn't exactly have a [nice manual like SLIME](http://common-lisp.net/project/slime/doc/slime.pdf) does, nor have I seen [exponents](http://planet.lisp.org/) like I have from the Lisp community.

But, browsing the docstrings it appears that python-mode offers rudimentary support for dynamic programming. It can run python in a subshell, and send strings to that shell. Unfortunately the listener is unmodified from what python gives you: no highlighting, no indentation, nothing. The subshell doesn't even know it's python that it talks to; it just recognizes the `>>>` and `...`.

There is some support for the python debugger, pdb, but I'm not sure how to get to it. It just recognizes the strings that pdb outputs.

There is supporting for reloading modules as well, but again it's quite primitive. All in all python-mode appears to be a start at dynamic programming but it does not approach SLIME. (This is an opportunity, not a condemnation.)

#### Coda

I wrote this article for a couple of reasons. One was for myself, to make sure that I understood everything that SLIME does. Another was to raise awareness of the integration possibilities between editors and dynamic languages, in the hopes that in a couple years I won't be using the same environment I'm using now.

I'm particularly interested in Emacs integration with Scheme and Python. There does exist a port of SLIME to Scheme48. I suspect that for languages not in the Lisp family, a fork or a "port of ideas" would be the best that can be done.

Comments welcome via e-mail or trackback, especially regarding other ways of programming dynamic languages. wingo at pobox dot com.

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [object closure and the negative specification](/archives/2008/04/22/object-closure-and-the-negative-specification)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)

### 2 responses

1. [programming musings » Blog Archive » Slime, a review by andy wingo](http://jaortega.wordpress.com/2006/01/05/slime-a-review-by-andy-win) says:[6 January 2006 3:11 AM](#32)

   \[...\] Andy has posted a detailed review o slime, showing some of the features that make it such an awesome development environment. It includes some comparisons with Python, that may be useful to those of you not familiar with Lisp. Technorati Tags Start Tags: lisp, slime Technorati Tags End \[...\]

2. [bang bang -- andy wingo](http://wingolog.org/?p=144) says:[20 January 2006 5:12 AM](#33)

   \[...\] Thanks to those that wrote mails regarding the slime writeup. Juho Snellman notes that Common Lisp does not actually specify a profiling interface, although most implementations provide an instrumenting profiler. It’s mainly SBCL that provides a statistical provider. Stephen Thorne writes that vim also has decent completion support via C-n, and there is another key that can complete entire lines (:help completion allegedly\[0\] has the details). \[...\]

Comments are closed.
