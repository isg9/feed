---
title: Live CSS Interaction with Skewer
url: https://nullprogram.com/blog/2013/01/24/
published: "2013-01-24T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/01/24/
---

# Live CSS Interaction with Skewer

## [Live CSS Interaction with Skewer](/blog/2013/01/24/)

January 24, 2013

nullprogram.com/blog/2013/01/24/

This evening [Skewer](/blog/2012/10/31/) gained support for live CSS. When editing CSS code, you can send your rules and declarations from the editing buffer to be applied in the open page in the browser. It makes experimenting with CSS really, really easy. The functionality is exposed through the familiar interaction keybindings, so if you’re already familiar with other Emacs interaction modes ( [SLIME](/blog/2010/01/15/), [nREPL](/blog/2013/01/07/), Skewer, [Geiser](http://www.nongnu.org/geiser/), Emacs Lisp), this should feel right at home.

To provide the keybindings in css-mode there’s a new minor mode, skewer-css-mode. CSS “expressions” are sent to the browser through the communication channel already provided by Skewer. It’s essentially an extension to Skewer: it could have been created without making any changes to Skewer itself.

Unfortunately Emacs’ css-mode is nowhere near as sophisticated as js2-mode — which reads in and exposes a full JavaScript AST. I had to write my own very primitive CSS parsing routines to tease things apart. It should generally be able to parse declarations and rules reasonably no matter how it’s indented, but it’s not very good at navigating *around* comments, especially when they contain CSS syntax. If I find a way to parse CSS more easily sometime I’ll see about fixing it, but it’s plenty good enough for now.

To “evaluate” the CSS, the code is simply dropped into the page as a new ``
