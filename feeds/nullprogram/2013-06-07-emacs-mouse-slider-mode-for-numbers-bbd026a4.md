---
title: Emacs Mouse Slider Mode for Numbers
url: https://nullprogram.com/blog/2013/06/07/
published: "2013-06-07T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/06/07/
---

# Emacs Mouse Slider Mode for Numbers

## [Emacs Mouse Slider Mode for Numbers](/blog/2013/06/07/)

June 07, 2013

nullprogram.com/blog/2013/06/07/

One of my regular commenters, and as of recently co-worker, Ahmed Fasih, sent me a video, [Live coding in Lua](http://youtu.be/FpxIfCHKGpQ). The author of the video added support to his IDE for scaling numbers in source code by dragging over them with the mouse. This feature was directly inspired by [Bret Victor](http://worrydream.com/#), a user interface visionary, probably best introduced through his presentation [Inventing on Principle](http://youtu.be/PUv66718DII).

I think Bret’s interface ideas are interesting and his demos very impressive. However, I feel they’re *too* specialized to generally be very useful. [Skewer](https://github.com/skeeto/skewer-mode) suffers from the same problem: in order to truly be useful, programs need to be written in a form that expose themselves well enough for Skewer to manipulate at run-time. Some styles of programming are simply better suited to live development than others. This problem is amplified in Bret’s case by the extreme specialty of the tools. They’re fun to play with, and probably great for education, but I can’t imagine any time I would find them useful while being productive.

Anyway, Ahmed wanted to know if it would be possible to implement this feature in Emacs. I said yes, knowing that Emacs ships with [artist-mode](http://www.emacswiki.org/emacs/ArtistMode), where the mouse can be used to draw with characters in an editing buffer. That’s proof that Emacs has the necessary mouse events to do the job. After spending a couple of hours on the problem I was able to create a working prototype: **mouse-slider-mode**.

- [https://github.com/skeeto/mouse-slider-mode](https://github.com/skeeto/mouse-slider-mode)

Demo video requires HTML5 with WebM support.

It’s a bit rough around the edges, but it works. When this minor mode is enabled, right-clicking and dragging left or right on any number will decrease or increase that number’s value. More so, if the current major mode has an entry in the `mouse-slider-mode-eval-funcs` alist, as the value is scaled the expression around it is automatically evaluated in the live environment. The documentation shows how to enable this in js2-mode buffers using skewer-mode. This is actually a step up from the other, non-Emacs implementations of this mouse slider feature. If I understood correctly, the other implementations re-evaluate the entire buffer on each update. My version only needs to evaluate the surrounding expression, so the manipulated code doesn’t need to be so isolated.

There is one limitation that cannot be fixed using Elisp. If the mouse exits the Emacs window, Elisp stops receiving valid mouse events. Number scaling is limited by the width of the Emacs window. Fixing this would require patching Emacs itself.

This is purely a proof-of-concept. It’s not installed in my Emacs configuration and I probably won’t ever use it myself, except to show it off as a flashy demo with an HTML5 canvas. If anyone out there finds it useful, or thinks it could be better, go ahead and adopt it.

- [emacs](/tags/emacs/)
- [media](/tags/media/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs%20Mouse%20Slider%20Mode%20for%20Numbers) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs+Mouse+Slider+Mode+for+Numbers).

« [A Handy Emacs Package Configuration Macro](/blog/2013/06/02/)

» [Long Live WebGL](/blog/2013/06/10/)
