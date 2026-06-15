---
title: Magick Thumbnails
url: https://nullprogram.com/blog/2009/12/21/
published: "2009-12-21T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/12/21/
---

# Magick Thumbnails

## [Magick Thumbnails](/blog/2009/12/21/)

December 21, 2009

nullprogram.com/blog/2009/12/21/

For a long time I couldn't figure out how to make decent thumbnails with [ImageMagick](http://www.imagemagick.org/script/index.php). Specifically, I wanted to create uniform sized thumbnails from arbitrary images. Over the weekend I came across the [ImageMagick Examples page](http://www.imagemagick.org/Usage/thumbnails/#cut), which shows exactly how to do this. Here's the command for a 150x150 thumbnail,

```
convert orig.jpg -thumbnail 150x150^ -gravity center \
        -extent 150x150 thumb.jpg

```

It cuts out the largest square possible from the center of the image and resizes that to 150x150. This capability has actually only been available for 2 years now! It wasn't there last time I needed it.

I can think of one way to improve it: instead of selecting the center, it selects the area with the highest information density. This could be measured by edge detection, corner detection, or some other statistical method. It would be selected by changing the gravity option to, say, "entropy".

I'm listing this here mostly for my own future reference. :-)

- [trick](/tags/trick/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Magick%20Thumbnails) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Magick+Thumbnails).

« [Game of Life in Java](/blog/2009/12/13/)

» [Setting up a Common Lisp Environment](/blog/2010/01/15/)
