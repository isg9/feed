---
title: colophonwards — wingolog
url: https://wingolog.org/archives/2023/12/05/colophonwards
published: "2023-12-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/12/05/colophonwards
---

# colophonwards — wingolog

## [colophonwards](/archives/2023/12/05/colophonwards)

5 December 2023 11:36 AM

- [meta](/tags/meta)
- [tekuti](/tags/tekuti)
- [css](/tags/css)
- [grid layout](/tags/grid%20layout)
- [fonts](/tags/fonts)
- [typography](/tags/typography)

A brief meta-note this morning: for the first time in 20 years, I finally got around to updating the web design of [wingolog.org](https://wingolog.org) recently and wanted to share a bit about that.

Back when I made [the initial wingolog
design](http://web.archive.org/web/20041230062546/http://wingolog.org/), I was using the then-brand-new Wordpress, [Internet Explorer
6](https://en.wikipedia.org/wiki/Internet_Explorer_6) was the most common web browser, CSS wasn’t very good, the Safari browser had just made its first release, smartphones were yet to be invented, and everyone used low-resolution CRT screens. The original design did use CSS instead of tables, thankfully, but it was very narrow and left a lot up to the user agent (notably font choice and size).

These days you can do much better. Even HTML has moved on, with [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/article)

and [`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/aside)[`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/aside) and [`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/section)[`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/section) elements. CSS is powerful and interoperable, with grid layout and media queries and `:has()` and `:is()` and all kinds of fun selectors. And, we have web fonts.

I probably would have stuck with the old design if it were readable, but with pixel counts growing, the saturated red bands on the sides flooded the screen, leaving the reader feeling like they were driving into headlights in the rain.

Anyway, the new design is a bit more peaceful, I hope. Feedback welcome.

I’m using grid layout, but not in the way that I thought I would. From reading the documentation, I had the impression that the element with `display: grid` would be a kind of flexible corkboard which could be filled up by any child element. However, that’s not quite true: it only works for direct children, which means your HTML does have to match the needs of the grid. Grandchildren can take their rows and columns from grandparents via `subgrid`, but only really display inside themselves: you can’t pop a grandkid out to a grandparent grid area. (Or maybe you can! It’s a powerful facility and I don’t claim to fully understand it.)

Also, as far as I can tell there is no provision to fill up one grid area with multiple children. Whereas I thought that on the root page, each blog entry would end up in its own grid area, that’s just not the case: you put the ``

(another new element!) in a grid area and let it lay out itself. Fine enough.

I would love to have [proper
side-notes](https://meyerweb.com/eric/thoughts/2023/09/12/nuclear-anchored-sidenotes/), and I thought the grid would do something for me there, but it seems that I have to wait for CSS anchor positioning. Until then you can use `position: absolute` tricks, but side-notes may overlap unless the source article is careful.

For fonts, I dearly wanted proper fonts, but I was always scared of the [flash of invisible
text](https://fonts.google.com/knowledge/glossary/foit). It turns out that with `font-display: swap` you can guarantee that the user can read text if for some reason your fonts fail to load, at the cost of a later layout shift when the fonts come in. At first I tried [Bitstream
Charter](https://practicaltypography.com/charter.html) for the body typeface, but I was unable to nicely mix it with [Fira
Mono](https://bboxtype.com/typefaces/FiraSans/#!layout=specimen) without line-heights getting all wonky: a ``` tag on a line would make that line too high. I tried all kinds of adjustments in the @font-face but finally decided to follow my heart and buy a font. Or two. And then sheepishly admit it to my spouse the next morning. You are reading this in Valkyrie, and the headings are Hermes Maia. I’m pretty happy with the result and I hope you are too. They are loaded from my server, to which the browser already has a TCP and TLS connection, so it would seem that the performance impact is minimal.`

`Part of getting performance was to inline my CSS file into the web pages produced by the blog software, allowing the browser to go ahead and lay things out as they should be without waiting on a chained secondary request to get the layout.`

`Finally, I did finally break down and teach my blog software’s marxdown parser about “smart quotes” and em dashes and en dashes. I couldn’t make this post in good faith without it; “the guy yammers on about web design and not only is he not a designer, he uses ugly quotes”, &c, &c...`

`Finally finally, some recommendations: I really enjoyed reading Erik Spiekermann’s Stop Stealing Sheep, 4th ed. on typography and type, which led to a raft of book purchases. Eric Meyer and Estelle Weyl’s CSS: The Definitive Guide was very useful for figuring out what is possible with CSS and how to make it happen. It’s a guide, though, and is not very opinionated; you might find Matthew Butterick’s Practical Typography to be useful if you are looking for pretty-good opinions about what to make.`

`Onwards and upwards!`

## `related articles`

- `service update`
- `it's probably spam`
- `the merry month of ma`
- `meta: per-tag feeds`
- `meta data`
- `metablog`

### `One response`

1. `Liudvikas says:18 December 2023 5:26 PM`

   `Very appreciated update! Previous style was a nightmare on the eyes`

`Comments are closed.`
