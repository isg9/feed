---
title: font-size-adjust Is Useful
url: https://matklad.github.io/2025/07/16/font-size-adjust.html
published: "2025-07-16T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2025/07/16/font-size-adjust.html
---

# font-size-adjust Is Useful

Jul 16, 2025

In this article, I will describe a recent addition to CSS, the `font-size-adjust` property. I am also making a bold claim that everyone in the world misunderstands the usefulness of this property, including [Google](https://web.dev/blog/font-size-adjust), [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size-adjust), and [CSS Specification itself](https://drafts.csswg.org/css-fonts-4/#propdef-font-size-adjust). (Just to clarify, no, I am not a web designer and I have no idea what I am talking about).

Let’s start with oversimplified and incorrect explanation of `font-size` (see [https://tonsky.me/blog/font-size/](https://tonsky.me/blog/font-size/) for details). Let’s say you specified `font-size: 96px`. What does that mean? First, draw a square 96 pixels high:

Then, draw a letter “m” somewhere inside this box:

m

This doesn’t make sense? I haven’t told you how large the letter m should be? Tiny? Huge? Well, sorry, but that’s really how font size works. It’s a size of the box around the glyph, not the size of the glyph. And there isn’t really much consistency between the fonts as to how large the glyph itself should be. Here’s a small “x” in the three fonts used on my blog at 48px font size:

xxx

They are quite different! And this is where `font-size-adjust` comes in. If I specify `font-size-adjust: ex-height 0.5`, I ask the browser to scale the font such that the letter “x” is exactly half of the box. This makes the fonts comparable:

xxx

## [Me vs. Everyone](\#Me-vs-Everyone)

Now, the part where I foolishly disagree with the world! The way this property is described in [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size-adjust) and elsewhere is as if it only matters for the font fallback. That is, if you have `font-family: Futura, sans-serif`, one potential problem could be that the fallback sans-serif font on the user’s machine will have very different size from Futura. So, the page could look very differently depending on whether fallback kicks in or not (and fallback can kick in *temporarily*, while the font is being loaded). So, the official guideline is, roughly,

> When using font fallback, find a value of `font-size-adjust` that makes no change for the first font of the fallback stack.

I don’t find this to be a particularly compelling use-case! Make sure to vendor the fonts used, specify `@font-face` inline in a ` `

`

```
          You are supposed to use coreutils to solve this problem.
          You are supposed to use coreutils to solve this problem.
          You are supposed to use coreutils to solve this problem.
          You are supposed to use coreutils to solve this problem.


          You are supposed to use coreutils to solve this
          problem.
          You are supposed to use coreutils to solve this
          problem.
          You are supposed to use coreutils to solve this
          problem.
          You are supposed to use coreutils to solve this
          problem.

      `
```

`

```
        .bonus-content p { margin-bottom: 24px; }
```

.bonus-content p, .bonus-content code { font-size: 16px; line-height: 24px; } .bonus-content { background-image: repeating-linear-gradient( transparent 0 23px, #ba3925 23px 24px ); }

```
        In the first paragraph, each line is indeed 24 pixels high, but in
        the second paragraph each line is slightly larger, despite the
        line-height being set to 24px explicitly. How can this be? The full
        answer is in:


        https://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align


        The TL;DR is that line-height doesn’t actually set
        the the height of the line (who would have thought!). Instead, it
        sets the height of each individual span of the text on the line. So,
        both “supposed” and “coreutils” have a 24
        pixels high box around them. But because relative position of glyphs
        inside the em-box is different between the two fonts, the boxes are
        shifted relative to each other to align the baselines. You can see
        that by adding vertical-align: bottom:


        You are supposed to use coreutils to solve this
        problem.
        You are supposed to use coreutils to solve this
        problem.
        You are supposed to use coreutils to solve this
        problem.
        You are supposed to use coreutils to solve this
        problem.


        If we align the boxes, than baselines are not aligned. It follows
        that when we align the baselines, the boxes are misaligned, and the
        line-height ends up larger than the height of any box, because boxes
        stick out!


        I don’t know a great solution here. A hack is to say something
        like
        p > code { line-height: 0px;
            }
        such that text set in monospace font doesn’t affect line height
        calculation. Counter-intuitively, this will work even if the line is
        entirely monospace (see strut in the abovelinked
        article).

    ``
  ``
````
