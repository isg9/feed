---
title: oversized crania — wingolog
url: https://wingolog.org/archives/2004/04/16/16-apr-2004
published: "2004-04-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/04/16/16-apr-2004
---

# oversized crania — wingolog

## [oversized crania](/archives/2004/04/16/16-apr-2004)

16 April 2004 7:39 PM

- [random](/tags/random)
- [scheme](/tags/scheme)
- [music](/tags/music)

**lisp and free software**

Eric Raymond says that [learning lisp will make you a better programmer](http://www.catb.org/~esr/faqs/hacker-howto.html#skills1). I can't speak about you, of course, but it's a true statement for me. I see different possibilities and patterns now. But the lisp world is a bizarre anti-universe.

I read all of [paul graham's essays](http://www.paulgraham.com/articles.html). As a schemer I identify with a lot of his lisp propaganda. But one of his (stranger) theses is that lisp people have enormous brains, which leads to the unstated corrolary that they rarely re-use work from other people. That's [occasionally a valid thesis](http://joelonsoftware.com/articles/fog0000000007.html), but it leads people to write their own [scheme](http://librep.sourceforge.net/) [implementations](http://mail.gnu.org/archive/html/gnu-arch-users/2003-11/msg00470.html), [reject the idea of having a common library of code](http://mail.gnu.org/archive/html/guile-user/2004-01/msg00190.html), write their own [personal bindings to motif/gtk](http://ccrma-www.stanford.edu/software/snd/snd/libxm.html), write their [own soundfile library](http://ccrma-www.stanford.edu/software/snd/snd/sndlib.html), etc. Gah, no more links!

This is less of a problem in other languages for two reasons: one, other languages can't have dialects, because they can't implement new syntax. (Parenthetically, how do you all survive without macros?) Often their language is the de facto language specification. Two, there's something about lisp that makes hackers want to write everything they do in lisp, and in their dialect to boot.

I write this as someone who just discovered lisp, and fell in love with it. I wrote a synthesizer, but I used gstreamer, libsndfile, ladspa plugins, gtk, etc. instead of [rolling my own everything](http://ccrma-www.stanford.edu/software/clm/). I love this gnomey aesthetic that we're developing. Software should be for everyone, not just died-in-the-wool UNIX hackers or those that shell out money to pay for it. How can you get there from here if you don't write for people?

And now, the caveats. [Bill Schottstaedt](http://mstation.org/schottstaedt.html), aside from being a fucking genius, also makes music with his work. Dogfooding indeed. Folks at [CCRMA](http://ccrma.stanford.edu/) teach some [interesting](http://ccrma-www.stanford.edu/courses/220a/) [courses](http://ccrma-www.stanford.edu/courses/220b/) about music and free software, and even offer redhat [packages-for-the-people](http://ccrma-www.stanford.edu/planetccrma/software/) at the CCRMA web site. And the software itself is becoming more free, but what's free software without free documenation? How can anyone from Namibia get, for example, a copy of Computer Music Journal?

**more lisp music**

I've been working in isolation for the past year, trying to reimplement SuperCollider 2 as free software. (SC3 is free software, but with no ui, and it's substantially different from SC2). It's only now that I started to do research into other computer music systems (silly me). I only really knew about CSound. It turns out there's a lot of stuff out there -- Common Lisp Music is the most interesting to me, and it turns out I've implemented a lot of similar ideas (inspired by SuperCollider's docs). It's good to have a milieu.

(An aside: Common Lisp Music actually generates the audio output with Lisp code, but it compiles the Lisp to C. Crazy!)

Unfortunately, when I say "out there", some of it really is out there. Confined to academic environments. Non-free. (A lot of the lisp projects use the commercial Allegro Common Lisp.) Not on the web. It's a cultural problem of navel-gazing / as-yet-unenlightened academics working on funded projects that see their work not as a contribution to society but a contribution to academia.

I'm pretty passionate about this -- when the means for artistic production are withheld from we the masses, we lose. We lose as spectators, not being able to listen/see/experience the interesting shit that could be made. We lose as potential artists, not being able to create, experiment, and maybe create something worthwhile. We have to do more to bring the possibilities of being an actor (rather than a consumer of entertainment) to everyone.

**dreams**

Strange things in the night -- basketball (I couldn't pass properly), salamanca (I got lost in the outskirts), bike rides (I crashed), and old friends (didn't quite connect). Whenever I tried to speak spanish, oshiwambo came out. Periodically woken up by rain on the tin roof. There's meaning there somewhere. I feel rested now.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)
- [radical adults](/archives/2006/12/15/radical-adults)
- [bang bang](/archives/2006/01/20/bang-bang)
- [Collected works](/archives/2005/09/26/collected-works)
- [Precipitation](/archives/2004/08/31/precipitation)

Comments are closed.
