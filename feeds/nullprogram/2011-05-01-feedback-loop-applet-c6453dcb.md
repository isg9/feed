---
title: Feedback Loop Applet
url: https://nullprogram.com/blog/2011/05/01/
published: "2011-05-01T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2011/05/01/
---

# Feedback Loop Applet

## [Feedback Loop Applet](/blog/2011/05/01/)

May 01, 2011

nullprogram.com/blog/2011/05/01/

*Update June 2014*: This [was ported to WebGL](/blog/2014/06/21/) and greatly improved.

- [https://github.com/skeeto/Feedback.git](https://github.com/skeeto/Feedback.git)

I was watching the BBC’s *The Secret Life of Chaos*, which is a very interesting documentary about chaos, [fractals](/blog/2007/10/01/), and emergent behavior. There is [a part where emergent behavior is demonstrated using
a video camera feedback loop](magnet:?xt=urn:btih:80e59413ca2b46e74f4a7572366a4a7de9b3e096) (35 minutes in). A camera, pointed at a projector screen, has it’s output projected onto that same screen. A match is lit, moved around a bit, and removed from the camera’s vision. At the center of the camera’s focus a pattern dances around for awhile in an unpredictable pattern, as the pattern is fed back into itself.

That’s the key to fractals and emergent behavior right there: a feedback loop. Apply some simple rules to a feedback loop and you might have a fractal on your hands. More examples,

- [How to make fractals without a computer](http://www.youtube.com/watch?v=Jj9pbs-jjis)
- [video feedback experiment 4](http://www.youtube.com/watch?v=xzJVbmqcj7k)

I was inspired to simulate this with software (and I [passed that on to
Gavin too](http://devrand.org/view/emergentFeedback)). Take an image, rescale it, rotate it, compose it with itself, repeat.

Here are some images I created with it.

![](/img/feedback/dense-tunnel.png)![](/img/feedback/single-spiral.png)![](/img/feedback/sun2.png)![](/img/feedback/jagged-spiral.png)

You can interact with the mouse — like the lit match. And that really is a feedback loop. In this video, you can see the mouse hop travel down through the iterations.

Unfortunately I didn’t seem to be able to achieve emergent behavior. The image operators too aggressively blur out fine details way down in the center. Bit that’s fine: it turned out more visually appealing than I expected!

Here’s a gallery of image captures from the applet. To achieve some of the effects try adjusting the rotation angle (press r or R) and scale factor (press g or G) while running the app. A couple of these were made using an experimental fractal-like image operator that can only be turned on in the source code.

[![](/img/feedback/fractal-thumb.png)](/img/feedback/fractal.png)[![](/img/feedback/gimpy-thumb.png)](/img/feedback/gimpy.png)[![](/img/feedback/green-spots-thumb.png)](/img/feedback/green-spots.png)[![](/img/feedback/halo-thumb.png)](/img/feedback/halo.png)[![](/img/feedback/orange-star-thumb.png)](/img/feedback/orange-star.png)[![](/img/feedback/pink-spiral-thumb.png)](/img/feedback/pink-spiral.png)[![](/img/feedback/spin-blur-thumb.png)](/img/feedback/spin-blur.png)[![](/img/feedback/spin-spiral-thumb.png)](/img/feedback/spin-spiral.png)[![](/img/feedback/spiral-thumb.png)](/img/feedback/spiral.png)[![](/img/feedback/star-thumb.png)](/img/feedback/star.png)[![](/img/feedback/sun-thumb.png)](/img/feedback/sun.png)[![](/img/feedback/tunnel-thumb.png)](/img/feedback/tunnel.png)[![](/img/feedback/gear-thumb.png)](/img/feedback/gear.png)

- [java](/tags/java/)
- [interactive](/tags/interactive/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Feedback%20Loop%20Applet) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Feedback+Loop+Applet).

« [October's Chess Engine Gets Some Attention](/blog/2011/04/09/)

» [Infinite Parallax Starfield](/blog/2011/06/13/)
