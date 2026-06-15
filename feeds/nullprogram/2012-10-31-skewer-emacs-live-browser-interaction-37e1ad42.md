---
title: 'Skewer: Emacs Live Browser Interaction'
url: https://nullprogram.com/blog/2012/10/31/
published: "2012-10-31T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2012/10/31/
---

# Skewer: Emacs Live Browser Interaction

## [Skewer: Emacs Live Browser Interaction](/blog/2012/10/31/)

October 31, 2012

nullprogram.com/blog/2012/10/31/

Inspired by [Emacs Rocks! Episode 11](http://youtu.be/qwtVtcQQfqc) on [swank-js](https://github.com/swank-js/swank-js), I spent the last week writing a new extension to Emacs to improve support for web development. It’s called [Skewer](https://github.com/skeeto/skewer-mode) and it allows you to interact with a browser like you would an inferior Lisp process. It’s written in pure Emacs Lisp, operates as a servlet for [my Elisp webserver](/blog/2009/05/17/), and **requires no special support from your browser or any other** **external programs**, making it portable and very easy to set up.

### Repository

- Available on [MELPA](http://melpa.milkbox.net/)

- [https://github.com/skeeto/skewer-mode](https://github.com/skeeto/skewer-mode)

### Demo

(No audio.)

[YouTube video](http://youtu.be/4tyTgyzUJqM)

The video also [on YouTube](http://youtu.be/4tyTgyzUJqM).

It works a little bit like [impatient-mode](http://50ply.com/blog/2012/08/13/introducing-impatient-mode/). First, the browser makes a long poll to Emacs. When you’re ready to send code to the browser to evaluate, Emacs wraps the expression in a bit of JSON and sends it to the browser. The browser responds with the result and starts another long poll.

As such, the browser doesn’t need to do anything special to support Skewer. If it can run jQuery, it can be skewered. I’ve tested it and found it working successfully on the latest versions of all the major browsers, including *you-know-who*.

To properly grab expressions around/before the point I’m using the *amazing* [js2-mode](https://github.com/mooz/js2-mode), originally written by the famous Steve Yegge. If you’re developing JavaScript you should be using this mode anyway! I thought I was clever with my [psl-mode](https://github.com/skeeto/psl-mode), writing my own full language parser. Steve Yegge did the same thing on a much larger scale three years ago with js2-mode. It includes an entire JavaScript 1.8 parser so the mode has *full* semantic understanding of the language. For Skewer, I use js2-mode’s functions to access the AST and extract complete, valid expressions.

### What’s wrong with swank-js?

Skewer provides nearly the same functionality as swank-js, a JavaScript back-end to SLIME. At a glance my extension seems redundant.

The problem with swank-js is the complicated setup. It requires a cooperating Node.js server, a particular version of SLIME, and a lot of patience. I could never get it working, and if I did I wouldn’t want to have to do all that setup again on another computer. In contrast, Skewer is just another Emacs package, no special setup needed. Thanks to `package.el` installing and using it should be no more difficult than installing any other package.

Most importantly, with Skewer I can capture the setup in my [.emacs.d repository](/blog/2011/10/19/) where it will automatically work across any operating system, so long as it has Emacs installed.

### Getting into JavaScript

I already used Skewer to develop a little [boids toy](https://github.com/skeeto/boids-js), which I’m using to demonstrate the mode (the video). Unlike my previous experiences in web development, this was extremely enjoyable — probably because it felt a lot like I was writing Lisp. And unlike any Lisp I’ve used so far, I had a canvas to draw on with my live code. That’s a satisfying tool to have.

Due to those prior poor experiences, I had avoided web development for a long time. But now that I have some decent tools configured I’m going to get into it more. In fact, I’ve decided I’m completely done with [writing Java applets](/toys/). Bounze will have been my last one.

This has become a pattern for me. When I want to start using a new language or platform I need to figure out a work-flow with Emacs. This involves trying out new modes, reading about how other people do it, and, ultimately, when I found out the existing stuff is inadequate I build my own extensions to create the work-flow I desire. I did this with [Java](/blog/2010/10/14/), recently with psl-mode (which was to be expected), and now web development.

In my recent proper introduction JavaScript in order to create and demo Skewer mode idiomatically, perhaps the most exciting discovery this past week was the JavaScript community itself. I’ve been mostly unaware of this community and taking my first steps into it has been enlightening.

JavaScript had a rough start. It was designed in a rush by developers who, at the time, didn’t quite understand the consequences of their design decisions, and later extended by similar people. The name of the language itself is evidence of this. Fortunately some really smart people jumped on board along the way (including Guy Steele of Lisp fame) and have tried to undo, or at least mitigate, the mistakes.

Due to the coarseness of the language, the JavaScript community is actually a lot like the Elisp community, but on a larger scale: there’s still a whole lot of frontier to explore and it’s pretty easy to make a noticeable splash.

Here’s to splashing!

- [emacs](/tags/emacs/)
- [javascript](/tags/javascript/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Skewer:%20Emacs%20Live%20Browser%20Interaction) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Skewer%3A+Emacs+Live+Browser+Interaction).

« [Debian Bugtracker Data](/blog/2012/10/15/)

» [JavaScript's Quirky eval](/blog/2012/11/14/)
