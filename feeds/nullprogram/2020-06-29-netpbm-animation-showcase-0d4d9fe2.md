---
title: Netpbm Animation Showcase
url: https://nullprogram.com/blog/2020/06/29/
published: "2020-06-29T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/06/29/
---

# Netpbm Animation Showcase

## [Netpbm Animation Showcase](/blog/2020/06/29/)

June 29, 2020

nullprogram.com/blog/2020/06/29/

Ever since I worked out [how to render video from scratch](/blog/2017/11/03/) some years ago, it’s been an indispensable tool in my software development toolbelt. It’s the first place I reach when I need to display some graphics, even if it means having to do the rendering myself. I’ve used it often in throwaway projects in a disposable sort of way. More recently, though, I’ve kept better track of these animations since some of them *are* pretty cool, and I’d like to look a them again. This post is a showcase of some of these projects.

Each project is in a ready-to-run state of compile, then run with the output piped into a media player or video encoding. The header includes the exactly commands you need. Since that’s probably inconvenient for most readers, I’ve included a pre-recorded sample of each. Though in a few cases, especially those displaying random data, video encoding really takes something away from the final result, and it may be worth running yourself.

The projects are not in any particular order.

### RANDU

[![](/img/showcase/randu.jpg)](https://nullprogram.com/video/?v=randu)

**Source**: [randu.c](https://github.com/skeeto/scratch/blob/master/animation/randu.c)

This is a little demonstration of the poor quality of the [RANDU
pseudorandom number generator](https://en.wikipedia.org/wiki/RANDU). Note how the source embeds a monospace font so that it can render the text in the corner. For the 3D effect, it includes an orthographic projection function. This function will appear again later since I tend to cannibalize my own projects.

### Color sorting

[![](/img/showcase/colorsort.jpg)](https://nullprogram.com/video/?v=colors-odd-even)

**Source**: [colorsort.c](https://github.com/skeeto/scratch/blob/master/animation/colorsort.c)

The original idea came from [an old reddit post](https://old.reddit.com/r/woahdude/comments/73oz1x/from_chaos_to_order/).

### Kruskal maze generator

[![](/img/showcase/animaze.jpg)](https://nullprogram.com/video/?v=kruskal)

**Source**: [animaze.c](https://github.com/skeeto/scratch/blob/master/animaze/animaze.c)

This effect was invented by my current [mentee student](/blog/2016/09/02/) while working on maze / dungeon generation late last year. This particular animation is my own implementation. It outputs Netpbm by default, but, for both fun and practice, also includes an entire implementation [in
OpenGL](/blog/2015/06/06/). It’s enabled at compile time with `-DENABLE_GL` so long as you have GLFW and GLEW (even on Windows!).

### Sliding rooks puzzle

[![](/img/showcase/rooks.jpg)](https://nullprogram.com/video/?v=rooks)

**Source**: [rooks.c](https://github.com/skeeto/scratch/blob/master/animation/rooks.c)

I wanted to watch an animated solution to [the sliding rooks
puzzle](https://possiblywrong.wordpress.com/2020/05/20/sliding-rooks-and-queens/). This program solves the puzzle using a bitboard, then animates the solution. The rook images are embedded in the program, compressed using a custom run-length encoding (RLE) scheme with a tiny palette.

### Glauber’s dynamics

[![](/img/showcase/magnet.jpg)](https://nullprogram.com/video/?v=magnet)

**Source**: [magnet.c](https://github.com/skeeto/scratch/blob/master/animation/magnet.c)

My own animation of [Glauber’s dynamics](http://bit-player.org/2019/glaubers-dynamics) using a totally unoriginal color palette.

### Fire

[![](/img/showcase/fire.jpg)](https://nullprogram.com/video/?v=fire)

**Source**: [fire.c](https://github.com/skeeto/scratch/blob/master/animation/fire.c)

This is the [classic Doom fire animation](https://fabiensanglard.net/doom_fire_psx/). I later [implemented it
in WebGL](/blog/2020/04/30/) with a modified algorithm.

### Mersenne Twister

[![](/img/showcase/mt.jpg)](https://nullprogram.com/video/?v=mt19937-shuffle)

**Source**: [mtvisualize.c](https://github.com/skeeto/scratch/blob/master/animation/mtvisualize.c)

A visualization of the Mersenne Twister pseudorandom number generator. Not terribly interesting, so I almost didn’t include it.

### Pixel sorting

[![](/img/showcase/pixelsort.jpg)](https://nullprogram.com/video/?v=pixelsort)

**Source**: [pixelsort.c](https://github.com/skeeto/scratch/blob/master/animation/pixelsort.c)

Another animation [inspired by a reddit post](https://old.reddit.com/r/generative/comments/9o1plu/generative_pixel_sorting_variant/). Starting from the top-left corner, swap the current pixel to the one most like its neighbors.

### Random walk (2D)

[![](/img/showcase/walkers.jpg)](https://nullprogram.com/video/?v=walk2d)

**Source**: [walkers.c](https://github.com/skeeto/scratch/blob/master/animation/walkers.c)

Another reproduction of [a reddit post](https://old.reddit.com/r/proceduralgeneration/comments/g49qwk/random_walkers_abstract_art/). This is recent enough that I’m using a [disposable LCG](/blog/2019/11/19/).

### Manhattan distance Voronoi diagram

[![](/img/showcase/voronoi.jpg)](https://nullprogram.com/video/?v=voronoi)

**Source**: [voronoi.c](https://github.com/skeeto/scratch/blob/master/animation/voronoi.c)

Another [reddit post](https://old.reddit.com/r/proceduralgeneration/comments/fuy6tk/voronoi_with_manhattan_distance_in_c/), though I think my version looks a lot nicer. I like to play this one over and over on repeat with different seeds.

### Random walk (3D)

[![](/img/showcase/walk3d.jpg)](https://nullprogram.com/video/?v=walk3d)

**Source**: [walk3d.c](https://github.com/skeeto/scratch/blob/master/animation/walk3d.c)

Another stolen idea personal take [on a reddit post](https://old.reddit.com/r/proceduralgeneration/comments/geka1q/random_walking_in_3d/). This features the orthographic projection function from the RANDU animation. Video encoding makes a real mess of this one, and I couldn’t work out encoding options to make it look nice, so this one looks a lot better “in person.”

### Lorenz system

[![](/img/showcase/lorenz.jpg)](https://nullprogram.com/video/?v=lorenz)

**Source**: [lorenz.c](https://github.com/skeeto/scratch/blob/master/animation/lorenz.c)

A 3D animation I adapted from the 3D random walk above, meaning it uses the same orthographic projection. I have [a WebGL version of this
one](/blog/2018/02/15/), but I like that I could do this in such a small amount of code and without an existing rendering engine. Like before, this is really damaged by video encoding and is best seen live.

Bonus: I made [an obfuscated version](https://gist.github.com/skeeto/45d825c01b00c10452634933d03e766d) just to show how small this can get!

- [c](/tags/c/)
- [media](/tags/media/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Netpbm%20Animation%20Showcase) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Netpbm+Animation+Showcase).

« [Latency in Asynchronous Python](/blog/2020/05/24/)

» [Exactly-Once Initialization in Asynchronous Python](/blog/2020/07/30/)
