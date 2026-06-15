---
title: cromulation — wingolog
url: https://wingolog.org/archives/2004/11/24/cromulation
published: "2004-11-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/11/24/cromulation
---

# cromulation — wingolog

## [cromulation](/archives/2004/11/24/cromulation)

24 November 2004 12:22 PM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

![meltdown](http://ambient.2y.net/wingo/wp-content/meltdown.jpg)

Well well, time passing without key-tapping on the weblog. Greetings, November, and see you later. Hardly knew you.

I fly out of this country three weeks from today. I have my itinerary, finally:

```
  Dec. 16: -> San Francisco
  Dec. 22: -> Lafayette, LA
  Dec. 25: -> Charlotte

```

but then...

```
  Jan 14: -> Barcelona

```

So, time back home is at a premium. Hence the US$2300 plane tickets. I'm excited to go -- there's none of the usual trepidation about the future, not knowing what's next. Time with friends and family is going to be really nice.

**state of the non-hack**

Lots of distractions these days, no time for programming. I still need to grade a couple of tests so that I can give my learners their final marks. Then there's the thanksgiving dinner this weekend -- 30+ people, at my house, two turkeys, six pies, stuffing and all of the normal accoutrements. Then there's the other thing -- the volunteers that will be replacing my group are in the are now, demanding attention and mostly deserving it. Interesting people, and new ones at that.

I put out a few releases of [guile-gnome](http://home.gna.org/guile-gnome/) recently. Nice to get some contributions from other folks. The issue as of late has been parallel installation and library versioning. Tricky, but I think we have the problem licked.

Got around to building guile 1.7 the other day. Threads in guile 1.7 are the platform's native threads, which is pthreads on unix. The only part that needs to run single-threaded is GC. Quite impressive -- I don't know of any other dynamic language that makes native threading this easy. Clue me in if you know of one. (Pthreads only, please :)

The photo above was posted with [photoblogger](http://ambient.2y.net/wingo/software/photoblogger/) (which has a [newer version](http://ambient.2y.net/wingo/pub/) out). I wouldn't post photos at all if I didn't write that app.

**the count**

22 days until landing in SF.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)
- [Precipitation](/archives/2004/08/31/precipitation)

Comments are closed.
