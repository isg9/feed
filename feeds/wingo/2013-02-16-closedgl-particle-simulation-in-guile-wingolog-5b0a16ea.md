---
title: closedgl particle simulation in guile — wingolog
url: https://wingolog.org/archives/2013/02/16/opengl-particle-simulation-in-guile
published: "2013-02-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/02/16/opengl-particle-simulation-in-guile
---

# closedgl particle simulation in guile — wingolog

## [closedgl particle simulation in guile](/archives/2013/02/16/opengl-particle-simulation-in-guile)

16 February 2013 7:20 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [closedgl](/tags/closedgl)
- [gl](/tags/gl)
- [ffi](/tags/ffi)

Hello, internets! Did you know that today marks [two years of Guile 2](http://lists.gnu.org/archive/html/guile-devel/2011-02/msg00173.html)? Word!

Guile 2 was a major upgrade to Guile's performance and expressiveness as a language, and I can say as a user that it is so nice to be able to rely on such a capable foundation for hacking the hack.

To celebrate, we organized a little birthday hack-feast -- a communal potluck of programs that Guilers brought together to share with each other. Like [last year](http://lists.gnu.org/archive/html/guile-user/2012-02/msg00048.html), we'll collect them all and publish a summary article on [Planet GNU](http://planet.gnu.org/) later on this week. This is an article about the hack that I worked on.

**figl**

[Figl](http://gitorious.org/guile-figl) is a Guile binding to OpenGL and related libraries, written using Guile's [dynamic FFI](http://www.gnu.org/software/guile/manual/html_node/Foreign-Function-Interface.html).

Figl is a collaboration between myself and Guile super-hacker Daniel Hartwig. It's pretty complete, with a generated binding for all of OpenGL 2.1 based on the specification files. It doesn't have a good web site yet -- it might change name, and probably we'll move to savannah -- but it does have great documentation in texinfo format, built with the source.

But that's not my potluck program. My program is a little demo particle simulation built on Figl.

[Click to download video](//wingolog.org/pub/figl-particle-effect.webm)

Source available [here](https://gitorious.org/guile-figl/guile-figl/blobs/master/examples/particle-system/vbo.scm). It's a fairly modern implementation using vertex buffer objects, doing client-side geometry updates.

To run, assuming you have some version of Guile 2.0 installed:

```
git clone git://gitorious.org/guile-figl/guile-figl.git
cd guile-figl
autoreconf -vif
./configure
make
# 5000 particles
./env guile examples/particle-system/vbo.scm 5000

```

You'll need OpenGL development packages installed -- for the `.so` links, not the header files.

**performance**

If you look closely on the video, you'll see that the program itself is rendering at 60 fps, using 260% CPU. (The CPU utilization drops as soon as the video starts, because of the screen capture program.) This is because the updates to the particle positions and to the vertices are done using as many processors as you have -- 4, in my case (2 real and 2 threads each). I've gotten it up to 310% or so, which is pretty good utilization considering that the draw itself is single-threaded and there are other things that need to use the CPU like the X server and the compositor.

Each particle has a position and a velocity, and is attracted to the center using a gravity-like force. If you consider that updating a particle's position should probably take about 100 cycles (for a square root, some divisions, and a few other floating point operations), then my 2.4GHz processors should allow for a particle simulation with 1.6 million particles, updated at 60fps. In that context, getting to 5K particles at 60 fps is not so impressive -- it's sub-optimal by a factor of 320.

This slowness is oddly exciting to me. It's been a while since I've felt Guile to be slow, and this is a great test case. The two major things to work on next in Guile are its newer register-based virtual machine, which should speed things up by a factor of 2 or so in this benchmark; and an ahead-of-time native compiler, which should add another factor of 5 perhaps. Optimizations to an ahead-of-time native compiler could add another factor of 2, and a JIT would probably make up the rest of the difference.

One thing I was really wondering about when making this experiment was how the garbage collector would affect the feeling of the animation. Back when I worked with the excellent folks at [Oblong](http://oblong.com/), I realized that graphics could be held to a much higher standard of smoothness than I was used to. Guile allocates floating-point numbers on the heap and uses the [Boehm-Demers-Weiser conservative collector](http://www.hpl.hp.com/personal/Hans_Boehm/gc/), and I was a bit skeptical that things could be smooth because of the pause times.

I was pleasantly surprised that the result was very smooth, without noticeable GC hiccups. However, GC is noticeable on an overloaded system. Since the simulation advances time by 1/60th of a second at each frame, regardless of the actual frame rate, differing frame processing times under load can show a sort of "sticky-zipper" phenomenon.

If I ever got to optimal update performance, I'd probably need to use a better graphics card than my Intel i965 embedded controller. And of course vertex shaders are really the way to go if I cared about the effect and not the compiler -- but I'm more interested in compilers than graphics, so I'm OK with the state of things.

Finally I would note that it has been pure pleasure to program with general, high-level macros, knowing that the optimizer would transform the code into really efficient low-level code. Of course I had to check the optimizations, with the `,optimize,` command at the REPL. I even had to add a couple of transformations to the optimizer at one point, but I was able to do that [without restarting the program](http://www.nongnu.org/geiser), which was quite excellent.

So that's my happy birthday dish for the Guile 2 potluck. More on other programs after the hack-feast is over; until then, see you in `#guile`!

## related articles

- [compost, a leaf function compiler for guile](/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [foreign interfaces](/archives/2007/05/24/foreign-interfaces)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 8 responses

1. [YurB](http://yurb.pp.ua/) says:[17 February 2013 9:50 AM](#b771f3ab18a9f88c74ed25dfdf89969099481c7c)

   I'm very excited to see opengl in guile! Maybe some day I'll try to learn opengl and guile together. Thanks for the work you did. But please, encode your videos into Ogg Theora or Webm, because I can't play your video in Firefox. Thanks again and good luck!

2. [YurB](http://yurb.pp.ua/) says:[17 February 2013 9:55 AM](#45f5dd02a4d7224ad3fe959c8a9d030ff9209f00)

   Oh, I'm sorry, a webm source is available in the html. But the browser still gives me an error: "No video with supported formand and MIME type found".

3. [YurB](http://yurb.pp.ua/) says:[17 February 2013 10:07 AM](#5574b922b681780049ff7f55b59837fbdae02daa)

   Very strange, the webm file is playing fine in vlc, but Firefox fails to play it (both locally and from your server). Chromium plays it without problems.

4. philn says:[17 February 2013 10:31 AM](#f57b6d806805021e0f6ac3ab2087fa9fc4a6dc53)

   The .webm file was generated by a buggy gnome-shell, the duration is set to 1 second or so :) I think this bug is fixed in a recent release.

5. philn says:[17 February 2013 10:34 AM](#2f65599a7a9ee56de44d3a6646fdf40b853b39f3)

   Never mind that comment, ignore me!

6. [wingo](http://wingolog.org/) says:[18 February 2013 9:12 AM](#3b1450b51c4cbce08aed3bdc807fbaf1442662d4)

   I'm not sure what the deal is with the webm video and Firefox. The video was transcoded from a gnome-shell webm screen capture using a simple vp8enc ! matroskamux GStreamer pipeline.

   I transcoded to Ogg as well, but it's strangely jerky. Probably something wrong with the original timestamps. But here's an Ogg link: [figl-particle-effect-small.ogv](http://wingolog.org/pub/figl-particle-effect-small.ogv).

7. paines says:[18 February 2013 10:32 AM](#07658dcc15c6058add2627ea7d717ffedf2bf224)

   Nice. Even under VMWare where GL is emulated in software I am getting around ~33fps in a small window(~70 frames drawn), and around ~22fps with 1920x1080(~45 frames drawn) on an i7, 2 cores for VBox, ~140% cpu.

8. Larry says:[19 February 2013 1:41 AM](#ef830a88dbab4701dea7fb67e82e03eb1e4d90bb)

   figl/contrib/packed-struct.scm:180:8: In procedure lp:

   figl/contrib/packed-struct.scm:180:8: In procedure +: Wrong type argument in position 1: Segmentation fault (core dumped)

Comments are closed.
