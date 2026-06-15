---
title: SXML and authoring — wingolog
url: https://wingolog.org/archives/2004/09/09/sxml-and-authoring
published: "2004-09-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/09/09/sxml-and-authoring
---

# SXML and authoring — wingolog

## [SXML and authoring](/archives/2004/09/09/sxml-and-authoring)

9 September 2004 12:14 PM

- [scheme](/tags/scheme)

### meta

Fixed a few broken links on the site recently. It seems a number of people still search for the [water management paper](http://ambient.2y.net/wingo/writings/water/html/) we did a while back.

The Apache Redirect and RedirectMatch directives have been crucial to the switch in site structure. I'm going to start a new campaign: *No more 404!*

### er... a little help here?

Four years ago, I put some [fortran code](http://ambient.2y.net/wingo/software/fortran/) on the web, hoping it would be useful. A few times a day, people would go to that page, try to download a file, get a 404, and just give up. Four years of that, and not once did someone email me to let me know. The web is a passive medium indeed.

### authoring in SXML

Lately I've taken to writing my web pages in SXML. It's quite useful as a templating engine. I can write [simple pages](http://ambient.2y.net/wingo/about/index.scm) and have them follow the look of the site.

SXML also lends itself well to programmatic processing. SXML provides a higher-order processing framework, `pre-post-order` from [`(sxml transform)`](http://ambient.2y.net/wingo/software/guile-lib/sxml-transform/), that is much easier and much more expressive than XSLT. With it, I can write in a [domain-specific language](http://ambient.2y.net/wingo/software/guile-lib/index.scm), then programmatically translate that more abstract language into plain HTML.

Finally, with SXML it's impossible to write malformed HTML. If the scheme interpreter can read the SXML, it is well-formed, and the translation process guarantees a well-formed output.

The system does have a drawback, though. Whenever a SCM file changes, I have to manually generate the HTML file. This would be made easier by a new Apache module to regenerate output as necessary. Then one could control the rebuilding rules via the standard .htaccess. Such a system would resemble a stripped-down version of `make`.

### quality of conversation

> Imagine the kind of conversation you would have with someone so far away that there was a transmission delay of one minute. Now imagine speaking to someone in the next room. You wouldn't just have the same conversation faster, you would have a different kind of conversation. In Lisp, developing software is like speaking face-to-face. You can test code as you're writing it. And instant turnaround has just as dramatic an effect on development as it does on conversation. You don't just write the same program faster; you write a different kind of program.

\-\- Paul Graham, [*On Lisp*](http://www.paulgraham.com/onlisp.html)

I've found this to be painfully true, as I stepped back from my wonderfully alive Guile development environment to hack up that wordpress-advogato plugin. I had to code over ssh from Africa to the US to edit code on the live site, because obviously it had to try to place the articles on advogato itself. That was excruciating -- type, and emacs changes a second later. Then to test it, I had to submit web forms back home, then check the different sites to see if I messed things up. I was shouting from another continent. Thus, the plugin sucks a lot.

I did fix up one thing recently, though. The output filters are applied before the data is sent to advogato. This should fix the paragraph problem I used to have.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

Comments are closed.
