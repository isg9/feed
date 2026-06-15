---
title: on yakshave, on color, on cosines, on glitchen — wingolog
url: https://wingolog.org/archives/2014/11/14/on-yakshave-on-color-on-cosines-on-glitchen
published: "2014-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/11/14/on-yakshave-on-color-on-cosines-on-glitchen
---

# on yakshave, on color, on cosines, on glitchen — wingolog

## [on yakshave, on color, on cosines, on glitchen](/archives/2014/11/14/on-yakshave-on-color-on-cosines-on-glitchen)

14 November 2014 4:49 PM

- [jpeg](/tags/jpeg)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [colorspaces](/tags/colorspaces)
- [yaks](/tags/yaks)
- [photos](/tags/photos)
- [dct](/tags/dct)
- [compression](/tags/compression)

[Hold on to your butts](http://butt.holdings/), kids, because this is epic.

**on yaks**

As in all great epics, our prideful, stubborn hero starts in a perfectly acceptable state of things, decides on a lark to make a small excursion, and comes back much much later to inflict upon you pictures from his journey.

So. I have a web photo gallery but I don't take many pictures these days. Dealing with photos is a bit of a drag, and the ways that are easier like Instagram or what-not give me the (peer, corporate, government: choose 3) surveillance hives. So, I had vague thoughts that I should update my web gallery. Yakpoint 1.

At the same time, my web gallery was written for `mod_python` on the server, and I don't like hacking in Python any more and kinda wanted to switch away from Apache. Yakpoint 2.

So I rewrote the server-side part in Scheme. (Yakpoint 3.) It worked fine but I found I needed the ability to get the dimensions of files on the server, so I wrote a quick-and-dirty JPEG parser. Yakpoint 4.

I needed EXIF data as well, as the original version displayed EXIF data, and for that I used a binding to `libexif` that I had written a few years ago when I thought about starting this project (Yakpoint -1). However I found some crashers in the library, because it had never really been tested in production, and instead of fixing them I said "what the hell, I'll just write an EXIF parser". (Yakpoint 5.) So I did and adapted the web gallery to use it (Yakpoint 6, for the adaptation.)

At this point, I looked back, and looked forward, and looked all around, and all was good, but what was with this uneasiness I was feeling? And indeed, I hadn't actually made anything better, and I wasn't taking more photos, and the workflow was the same.

I was also concerned about the client side of things, which was still in Python and using some breakage-prone legacy libraries to do the photo scaling and transformations and what-not, and relied on a desktop application ( [f-spot](http://f-spot.org/)) of dubious future. So I started to look at what it would take to port that script to Scheme (Yakpoint 7). Well it used some legacy libraries to copy files over SSH ( `gnome-vfs`; switching away from that would be Yakpoint 8) and I didn't want to make a Scheme [GIO](https://developer.gnome.org/gio/) binding (Yakpoint 9, narrowly avoided), and I then -- and then, dear reader -- so then I said "well WTF my caching story on the server is crap anyway, I never know when the sqlite database has changed or not so I never know what responses I can cache, what I really want is a functional datastore" (Yakpoint 10), which is what I have with [Git](//wingolog.org/archives/2008/04/12/git-a-transcriptional-database) and [Tekuti](//wingolog.org/projects/tekuti/) (Yakpoint of yore), and so why not just store my photos in Git like I do in Tekuti for blog posts and serve them from there, indexing as needed? Of course I'd need some other server software (Yakpoint of fore, by which I meantersay the future), but then I could just `git push` to update my photo gallery, and I wouldn't have to endure the horror that is [GVFS shelling out to `ssh` in a FUSE daemon](https://git.gnome.org/browse/gvfs/tree/daemon/gvfsbackendsftp.c#n1621) (Yakpoint of ne'er).

So. After mulling over these thoughts for a while I decided, during an autumnal walk on the Salève in which we had the greatest views of Mont Blanc everrrrr and yet where are the photos?, that really what I needed was new photo management software, not just a web gallery. I should be able to share photos from my phone or from my desktop, fix them up either place, tag and such, and OK woo hoo! Such is the future! And the present for many people? Thing is, I also needed good permissions management (Yakpoint what, 10 I guess?), because you know a dude just out of college is not the same as that dude many years later. Which means serving things over HTTPS ( [Yakpoints 11-47](http://wingolog.org/archives/2014/10/17/ffs-ssl)) in such a way that the app has some good control over who gets what.

Well. Anyway. My mind ran ahead, and runs ahead, and yet we haven't actually tasted the awesome sauce yet. So! The photo management software, whereever it lives, needs to rotate photos at least, and scale them down to a few resolutions. I smell a yak! I looked at `jpegtran` which can do some lossless rotations but it's not available as a library, which is odd; and really I don't like shelling out for core program functionality, because every time I deal with the file system it's the wild west of concurrent mutation. If naming things is one of the two hardest problems in computer science, the file system is the worst because you have to give a global name to every intermediate value.

At the same time to scale images, what was I to do? Make a binding to `libjpeg`? Well I started (Yakpoint 48) but for reals kids, `libjpeg` is not fun. It works great and is really clever but

1. it's approximately impossible to use from a dynamic ffi; you want a compiler to verify that you are using the right structure definitions

2. there has been an [inane ABI and format break imposed by the official IJG `libjpeg`](http://libjpeg-turbo.org/About/Jpeg-9) but which other implementations have not followed, but how could you know which one you are using?

3. the error handling facility encourages `longjmp` in C programs; somewhat terrifying

4. off-heap image manipulation libraries always interact poorly with GC, because the GC only sees the small pointer to the off-heap image, and so doesn't GC often enough

5. I have *zero* guarantee that `libjpeg` won't change ABI in weird ways, and I don't want to touch this software for the next 10 years

6. I want to do `jpegtran`-like lossless transformations, but that's not available as a library, and it's totes ridics that binding `libjpeg` does not help you out here

7. it's still an unsafe C library, battle-tested yes, but terrifyingly unsafe, and I'd be putting it on my server and who knows?

Friends, I arrived at the pasture, and I, I chose the yak less shaven. I took my lame JPEG parser and turned it into a full decoder (Yakpoint 49), realized it wasn't much more work to do an encoder (Yakpoint 50), and implemented the lossless transformations (Yakpoint 51).

**on haters**

Before we go on, I know some people would think "what is this kid about". I mean, custom gallery software, a custom JPEG library of all things, all bespoke, why don't you just use off-the-shelf solutions? Why aren't you *normal* and use a *normal language* and what about the *best practices* and where's your *business case* and I can't go on about this because there's a technical term for people that say this kind of thing and it's "hater".

Thing is, when did a hater ever make anything cool? Come to think of it, when did a hater make anything at all? In my experience the most vocal haters have nothing behind their names except a long series of pseudonymous rants in other people's comment boxes. So friends, in the [joyful spirit of earning-anew](//wingolog.org/archives/2014/08/27/a-wingolog-users-manual), let's talk about JPEG!

**on color**

JPEG is a funny thing. Photos are our lives and our memories, our first steps and our friends, and yet I for one didn't know very much about them. My mental model that "a JPEG is a rectangle of pixels" doesn't turn out to be quite right.

If you actually look in a normal JPEG, you see three planes of information. If I take this image, for example:

![](//wingolog.org/pub/capitalisme-cropped-stripped.jpeg)

If I decode it, actually I get three images. Here's the first one:

![](//wingolog.org/pub/capitalisme-cropped-0-stripped.jpeg)

This is just the greyscale version of the image. So, storytime! Remember black and white television? We had an old one that got moved around the house sometimes, like if Mom was working at something in the kitchen. We also had a color one in the living room, and you could watch one or the other and they showed the same stuff. Strange when you think about it though -- one being in color and the other not. Well it turns out that color was literally just added on, both historically and technically. The main broadcast was still in black and white, and then [in one part of the frequency band](https://en.wikipedia.org/wiki/NTSC#Color_encoding) there were separate color signals, which color TVs would pick up, mix with the black and white signal, and come out with color. Wikipedia notes that "color TV" was really just "colored TV", which is a phrase whose cleverness I respect. Big ups to the W P.

In the context of JPEG, this black-and-white signal is sometimes called "luma", but is more precisely called Y', where the "prime" (the apostrophe) indicates that the signal has gamma correction applied.

In the image above, I replaced the color planes (sometimes collectively called the "chroma") with zeroes, while losslessly keeping the luma. Below is the first color plane, with the Y' plane replaced with a uniform 50% luma, and the other color plane replaced with zeros.

![](//wingolog.org/pub/capitalisme-cropped-1-stripped.jpeg)

This color signal is technically known as CB, which may be very imperfectly understood as the bluish component of the color. Well the original image wasn't very blue, so we don't see very much here.

Indeed, our eyes have a harder time seeing differences in color than differences in intensity. Apparently this goes all the way down to biology -- we have more receptors in our eyes for "black and white" and fewer for color.

Early broadcasters took advantage of this difference in perception by actually devoting more bandwidth in their broadcasts to luma than to chroma; if you check the Wikipedia page you will see that the [area in the spectrum allocation devoted to color is much smaller than the area devoted to intensity](https://en.wikipedia.org/wiki/NTSC#Color_encoding). So it is in JPEG: the above image being half-width indicates that actually we're just encoding one CB sample for every two Y' samples.

![](//wingolog.org/pub/capitalisme-cropped-2-stripped.jpeg)

Finally, here we have the CR color plane, which can loosely be thought of as the "redness" of the image.

These test images and crops preserve the actual encoding of this photo as it came from my camera, without re-encoding. That's partly why there's not much interesting going on; with the megapixels these days, it's hard to fit much of anything in a few hundred pixels square. This particular camera is sub-sampling in the horizontal direction, but it's also common to subsample vertically as well, producing color planes that are half-width and half-height. In my limited investigations I have found that cameras tend to sub-sample just in the X direction, producing what they call [`4:2:2`](http://www.poynton.com/PDFs/Chroma_subsampling_notation.pdf) images, and that standard software encoders subsample in both, producing `4:2:0`.

Incidentally, properly scaling up the color planes is quite an irritating endeavor -- the standard indicates that the color is sampled between the locations of the Y' samples ("centered" chroma), but these images originally have EXIF data that indicates that the color samples are taken at the position of the first Y' sample ("co-sited" chroma). I'm pretty sure `libjpeg` doesn't delve into the EXIF to check this though, so it would seem that all renderings I have seen of these photos are subtly off.

But how do you get proper color out of these strange luma and chroma things? Well, the Y'CBCR colorspace is really just the same color cube as RGB, except rotated: the Y' axis traverses the diagonal from `(0, 0, 0)` (black) to `(255, 255, 255)` (white). CB and CR are perpendicular to that diagonal, pointing towards blue or red respectively. So to go back to RGB, you multiply by a matrix to rotate the cube.

[![](//wingolog.org/pub/y-cb-cr.gif)](https://commons.wikimedia.org/wiki/File:YCbCr.GIF)

It's not a very intuitive color system, as you can see from the images above. For one thing, at zero or full luma, the chroma axes have no meaning; black and white can have no hue. Indeed if you imagine trying to fit a cube corner-down into a similar-sized box, you end up either having empty space in the box, or you have to cut off corners from the cube, or both. Cut corners means that bits of the Y'CBCR signal are wasted; empty space means there are RGB colors that are not representable in Y'CBCR. I'm not sure, but I think both are true for the particular formulation of Y'CBCR used in JPEG.

There's more to say about color here but frankly I don't know enough to do so, even though I worked in digital video for many years. If this is something you are mildly interested in, I highly, highly recommend watching [Wim Taymans' presentation at this year's GStreamer conference](http://gstconf.ubicast.tv/videos/image-conversion/). He takes a look at color in video that is constructive, building up from biology through math to engineering. His is a principled approach rather than a list of rules. It really clarified a number of things for me (and opened doors to unknown unknowns beyond).

**on cosines**

Where were we? Right, JPEG. So the proper way to understand what JPEG is is to understand the encoding process. We've covered colorspace conversion from RGB to Y'CBCR and sub-sampling. Next, the image canvas is divided into equal-sized "macroblocks". (These are called "minimum coded units" (MCUs) in the JPEG context, but in video they are usually called macroblocks, and it's a better name.) Without sub-sampling, each macro-block will contain one 8-sample-by-8-sample block for each component (Y', CB, CR) of the image. In my images above, the canvas space corresponding to one chroma block is the space of two luma blocks, so the macroblocks will be 16 samples wide and 8 samples tall, and contain two Y' blocks and one each of CB and CR. If the image canvas can't be evenly divided into macroblocks, it is padded to fit, usually by duplicating the last column or row of samples.

Then to make a JPEG, each block is encoded separately, then the whole thing is just written out to a file, and you're done!

This description glosses over a couple of important points, but it's a good big-picture view to have in mind. The pipeline goes from RGB pixels, to a padded RGB canvas, to separate Y'CBCR planes, to a possibly subsampled set of those planes, to macroblocks, to encoded macroblocks, to the file. Decoding is the reverse. It's a totally doable, comprehensible thing, and that was one of the big takeaways for me from this project. I took photography classes in high school and it was really cool to see how to shoot, develop, and print film, and this is similar in many ways. The real "film" is raw-format data, which some cameras produce, but understanding JPEG is like understanding enlargers and prints and fixer baths and such things. It's smelly and dark but pretty cool stuff.

![](//wingolog.org/pub/test-strip.jpeg)

So, how do you encode a block? Well peoples, this is a kinda cool thing. Maybe you remember from some math class that, given n uniformly spaced samples, you can always represent that series as a sum of n cosine functions of equally spaced frequencies. In each litle 8-by-8 block, that's what we do: a "forward discrete cosine transformation" (FDCT), which is just multiplying together some matrices for every point in the block. The FDCT is completely separable in the X and Y directions, so the space of 8 horizontal coefficients multiplies by the space of 8 vertical coefficients at each column to yield 64 total coefficients, which is not coincidentally the number of samples in a block.

Funny thing about those coefficients: each one corresponds to a particular horizontal and vertical frequency. We can map these out as a space of functions; for example giving a non-zero coefficient to (0, 0) in the upper-left block of a 8-block-by-8-block grid, and so on, yielding a 64-by-64 pixel representation of the *meanings* of the individual coefficients. That's what I did in the test strip above. Here is the luma example, scaled up without smoothing:

![](//wingolog.org/pub/dct.png)

The upper-left corner corresponds to a frequency of 0 in both X and Y. The lower-right is a frequency of 4 "hertz", oscillating from highest to lowest value in both directions four times over the 8-by-8 block. I'm actually not sure why there are some greyish pixels around the right and bottom borders; it's not a compression artifact, as I constructed these DCT arrays programmatically. Anyway. Point is, your lover's smile, your sunny days, your raw urban graffiti, your child's first steps, all of these are reified in your photos as a sum of cosine coefficients.

The odd thing is that what is reified into your pictures isn't actually all of the coefficients there are! Firstly, because the coefficients are rounded to integers. Mathematically, the FDCT is a lossless operation, but in the context of JPEG it is not because the resulting coefficients are rounded. And they're not just rounded to the nearest integer; they are probably quantized further, for example to the nearest multiple of 17 or even 50. (These numbers seem exaggerated, but keep in mind that the range of coefficients is about 8 times the range of the original samples.)

The choice of what quantization factors to use is a key part of JPEG, and it's subjective: low quantization results in near-indistinguishable images, but in middle compression levels you want to choose factors that trade off subjective perception with file size. A higher quantization factor leads to coefficients with fewer bits of information that can be encoded into less space, but results in a worse image in general.

JPEG proposes a standard quantization matrix, with one number for each frequency (coefficient). Here it is for luma:

```
(define *standard-luma-q-table*
  #(16 11 10 16 24 40 51 61
    12 12 14 19 26 58 60 55
    14 13 16 24 40 57 69 56
    14 17 22 29 51 87 80 62
    18 22 37 56 68 109 103 77
    24 35 55 64 81 104 113 92
    49 64 78 87 103 121 120 101
    72 92 95 98 112 100 103 99))

```

This matrix is used for "quality 50" when you encode an 8-bit-per-sample JPEG. You can see that lower frequencies (the upper-left part) are quantized less harshly, and vice versa for higher frequencies (the bottom right).

```
(define *standard-chroma-q-table*
  #(17 18 24 47 99 99 99 99
    18 21 26 66 99 99 99 99
    24 26 56 99 99 99 99 99
    47 66 99 99 99 99 99 99
    99 99 99 99 99 99 99 99
    99 99 99 99 99 99 99 99
    99 99 99 99 99 99 99 99
    99 99 99 99 99 99 99 99))

```

For chroma (CB and CR) we see that quantization is much more harsh in general. So not only will we sub-sample color, we will also throw away more high-frequency color variation. It's interesting to think about, but also makes sense in some way; again in photography class we did an exercise where we shaded our prints with colored pencils, and the results were remarkable. My poor, lazy coloring skills somehow rendered leaves lifelike in different hues of green; really though, they were shades of grey, colored in imprecisely. "Colored TV" indeed.

With this knowledge under our chapeaux, we can now say what the "JPEG quality" setting actually is: it's simply that pair of standard quantization matrices scaled up or down. Towards "quality 100", the matrix approaches all-ones, for no quantization, and thus minimal loss (though you still have some rounding, often subsampling as well, and RGB-to-Y'CBCR gamut loss). Towards "quality 0" they scale to a matrix full of large values, for harsh quantization.

This understanding also explains those wavey JPEG artifacts you get on low-quality images. Those artifacts look like waves because they are waves. They usually occur at sharp intensity transitions, which like a cymbal crash cause lots of high frequencies that then get harshly quantized. Incidentally I suspect (but don't know) that this is the same reason that cymbals often sound bad in poorly-encoded MP3s, because of harsh quantization in the frequency domain.

Finally, the coefficients are written out to a file as a stream of bits. Each file gets a huffman code allocated to it, which ideally is built from the distribution of quantized coefficient sizes seen in all of the blocks of an image. There are usually different encodings for luma and chroma, to reflect their different quantizations. Reading and writing this bitstream is a bit of a headache but the algorithm is specified in the JPEG standard, and all you have to do is implement it. Notably, though, there is special support for encoding a run of zero-valued coefficients, which happens often after quantization. There are rarely wavey bits in a blue blue sky.

![](//wingolog.org/pub/test-strip-shuffled.jpeg)

**on transforms**

It's terribly common for photos to be wrongly oriented. Unfortunately, the way that many editors fix photo rotation is by setting a bit in the EXIF information of the JPEG. This is ineffectual, as web browsers don't look in the EXIF information, and silly, because it turns out you can losslessly rotate most JPEG images anyway.

Consider that the body of a JPEG is an array of macroblocks. To rotate an image, you just have to rearrange those macroblocks, then rearrange the blocks inside the macroblocks (e.g. swap the two Y' blocks in my above example), then transform the blocks themselves.

The lossless transformations that you can do on a block are transposition, vertical flipping, and horizontal flipping.

Transposition flips a block along its downward-sloping diagonal. To do so, you just swap the coefficients at `(u, v)` with the coefficients at `(v, u)`. Easy peasey.

Flipping is trickier. Consider the enlarged DCT image from above. What would it take to horizontally flip the function at `(0, 1)`? Instead of going from light to dark, you want it to go from dark to light. Simple: you just negate the coefficients! But you only want to negate those coefficients that are "odd" in the X direction, which are those coefficients whose column is odd. And actually that's all there is to it. Flipping vertically is the same, but for coefficients whose row is odd.

I said "most images" above because those whose size is not evenly divided by the macroblock size can't be losslessly rotated -- you will end up seeing some of the hidden data that falls off the edge of the canvas. Oh well. Most raw images are properly dimensioned, and if you're downscaling, you already have to re-encode anyway.

But that's just flipping and transposition, you say! What about rotation? Well it turns out that you can express rotation in terms of these operations: rotating 90 degrees clockwise is just a transpose and a horizontal flip (in that order). Together, flipping horizontally, flipping vertically, and transposing form a group, in the same way that [flipping and flopping form a group for mattresses](http://www.americanscientist.org/issues/id.3465,y.0,no.,content.true,page.1,css.print/issue.aspx). Yeah!

**on scheme**

I wrote this library in Scheme because that's my language of choice these days. I didn't run into any serious impedance mismatches; Guile has a generic multi-dimensional array facility that made it possible to express many of these operations as generic folds, unfolds, or maps over arrays. The huffman coding part was a bit irritating, but all in all things were pretty good. The speed is pretty bad, but I haven't optimized it at all, and it gives me a nice test case for the compiler. Anyway, it's been fun and it suits my needs. [Check out the project page if you're interested](https://gitorious.org/guile-jpeg/guile-jpeg). Yes, to shave a yak you have to get a bit bovine and smelly, but yaks live in awesome places!

Finally I will leave you with a glitch, one of many that I have produced over the last couple weeks. Comments and corrections welcome below. Happy hacking!

![](//wingolog.org/pub/police-car-cropped-stripped.jpeg)

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

### 5 responses

1. [Christian Schaller](http://www.linuxrising.org) says:[14 November 2014 5:29 PM](#dc1413cf633c097ce17b382b4d56091aaf41de9e)

   Haters will always hate :)

2. [Alaric Snell-Pym](http://www.snell-pym.org.uk/alaric/) says:[15 November 2014 2:15 PM](#98f6c77187fc123ee3869854828950ecc7060c2c)

   Interesting... I'm slowly working towards a photo management system for my wife and I to replace our aging PHP-based one; I'm planning on building it on top of my Ugarit content-addressed storage project! I was just going to shell out to jpegtran for the rotations (why won't people just hold cameras the right way up?!), but I might be talked into porting your Scheme to Chicken...

3. [John Cowan](http://www.ccil.org/~cowan) says:[22 November 2014 5:21 PM](#42c23253671b9ae2847049a2b21219bb43134c16)

   *more receptors in our eyes for "black and white" and fewer for color*

   That's literally true but doesn't mean what you think it means. The BW receptors (called "rods" from their shape) are only for perception in low light levels, and are completely saturated and ignored in daylight color vision. Our eyes actually do use RGB perception, not the color TV system.

4. [DevFactor](http://www.devfactor.net) says:[3 December 2014 2:00 AM](#faab8cbbc0d2a357be85a44eafaaf3d7ec5b2953)

   It's more complicated than just rods and cones. Remember, retina, rods, cones and fovea are all receptors.

5. [Paul Ciarlo](http://ronaldravin.com/) says:[30 April 2015 1:50 AM](#2b935203c614daaecb7772897d594c0d9d90abc0)

   Great read, thanks!! The article and the code :-) My scheme is a bit rusty. Can you do H.264 next? ;-)

Comments are closed.
