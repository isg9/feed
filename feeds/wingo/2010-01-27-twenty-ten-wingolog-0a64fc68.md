---
title: twenty ten — wingolog
url: https://wingolog.org/archives/2010/01/27/twenty-ten
published: "2010-01-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/01/27/twenty-ten
---

# twenty ten — wingolog

## [twenty ten](/archives/2010/01/27/twenty-ten)

27 January 2010 0:00 AM

- [guile](/tags/guile)
- [hack](/tags/hack)
- [scheme](/tags/scheme)
- [ffi](/tags/ffi)
- [profiling](/tags/profiling)

Sup tubes,

I'm feeling dandy as a dandy here, having just finished implementing a dynamic foreign function interface for Guile. By that I mean that you can call C procedures without writing any C code.

To be fair, most Schemes have this capability already, so Guile is playing catch-up here. But still, this is some sweetness. I can call functions with any number and type of arguments, including by-value struct args and return values, and things just work. It's nice, and liberating.

**intensification**

Continuing a series of things I'm told happen in France, I'm told that when you don't know the words to a song, and just make some stuff up, in French you're singing yogurt. *Yaourt*, rather. As in, *Teak me daou tou ze pair dais yaourt yoaurt yaourt*. I've always thought that Danone should host an international yogurt competition.

Anyway. nontechnical readers, feel free to hum the yogurt national anthem for a few paragraphs here, 'cause I'm going to talk about Guile a bit.

Guile! Hark, the beloved hack-country! Yes. We just made a release, did you know? [1.9.7](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=NEWS;h=dbe5b11f02c1126602388fa634974ea8cfa40a90;hb=a70c0ff578712ab8170aea0d2fb0d9b53cee8c5c), dontcha know. I'm particularly pleased that if you enter an expression at the REPL and something goes wrong, and you don't catch it, you are dropped into the debugger instead of being presented with a backtrace printout.

The debugger is totally silly at this point. You can't do much more than just print a backtrace. But you can parameterize the printing of that trace, print local variables for each frame ( `bt full`), inspect values, print with different widths...

That last bit is crucial. You typically don't want objects to print out in their entirety, because that could be quite long. So you truncate the output. But Murphy's law says that the part that gets truncated is always the part that you need! So actually, being able to specify the `#:width` is a big win.

Also you can profile expressions. Like this one, check it:

```
scheme@(guile-user)> ,profile (resolve-module '(gnome gtk)) #:hz 20
%     cumulative   self
time   seconds     seconds      name
 13.16      0.12      0.12  variable-bound?
 10.53      0.10      0.10  #
  7.89      2.80      0.07  vm-apply
  7.89      1.27      0.07  memq
  7.89      0.17      0.07  module-make-local-var!
[...]

```

In this case it reminded me that I had some bugs to fix, that things were taking about 10 times longer than they should have. But hey, that's what profilers are for, right?

One can trace, too:

```
scheme@(guile-user)> ,trace (fib 3)
(fib 3)
|(fib 2)
||(fib 1)
||1
||(fib 0)
||1
|2
|(fib 1)
|1
3

```

Sweet? \[Y/N\]

There are other lovelies in that release: the SSAX XML toolkit is now part of Guile, there's some unicode normalization improvements, some speedups, more compile warnings, and such. It was a lovely release. 1.9.8 will be even better!

**digression**

This stuff isn't always on my mind, you know. Most of the time, perhaps; but maybe in the same way that what you go to sleep thinking about is in your dreams, and in your mind vaguely in the morning. I gave my folks the (excellent) Settlers of Catan game this Christmas, with the Cities & Knights 5-6 player extension, and we all slept dreaming of sheep.

(Catan is a great game, for those few of you that haven't had the pleasure yet, and it appeals to most everyone. Call a game night, for great justice!)

Anyway. Probably what's most been on my mind in recent weeks is movement: to Paris and back (often), to the north of France for Christmas, to Bruges and Ghent for New Year's, to Lousiana just recently for a number of birthdays -- speaking of which: I am now 30. Trust my words no more!

Apart from that, time passes. A few months ago I planted the idea in my parents' mind that we were going to make or join a commune in a few years. At this point they're totally down. They've put off buying "old age insurance", or whatever you call it -- instead we younguns will take care of them. It's a retirement plan.

If you're reading this from anywhere that has roots, at any level, this won't surprise you at all. But the states is a different kettle of fish. At 13 they bussed me 30 miles away to secondary school; at 15 I went to (public) [boarding school](http://www.ncssm.edu/), 250 miles away; at 17 to university, at a similar distance. But the university bit was "close", family-wise. It's common to go 1000 miles away. Then when you graduate, it's common to go 1000 miles in a different direction, to find a job, and then likewise 5 years later when you change jobs. Myself, I left the states, and spent 8 years of the aughties abroad.

What I mean to say though is that you aren't expected to have roots in the states. I didn't live near my grandparents. Going home just now I didn't go to North Carolina, I went to Louisiana where my mom's family is.

These are many details, but it's all to say, the future is going to be like the past, but not like the recent past. It was really neat to hear my mom and dad and sisters and aunt all down with the idea of living together. Separate "houses", same place. I'm sure we can get some more too -- friends, partner's family, &c. Maybe we can even take over a town government. The times, they are a'rollin' round, round to a re-vision of what they were.

**nerd resolutions**

Readers: you really didn't want to know my personal resolutions, did you? Perhaps you did. Kind readers, you few.

Silent sufferers, the rest; but here is some nerd fodder, in the form of new year's resolutions:

- Switch to [Notmuch mail](http://notmuchmail.org/). Will require some kind of Gnus-like integration.

- Get a version of elisp running under Guile that's faster than Emacs' implementation, but still strictly compatible.

- Release Guile 2.0 in March, to wide acclaim.

- Go to FOSDEM, the European Lisp Conference, GUADEC, and one other conference. (More if you invite me to speak.)

- [Work](http://oblong.com/)-related: start working in a G-Speak environment by default (with G-Speak as the window manager).

- Start poking solutions to Free distributed web services. GNU has a huge role to play here, and I think Guile is the language/vm in which to implement the applications. More on this in the future.

- See if I can corral the necessary elements to get a working Guile-in-Emacs branch in Emacs' repository. A stretch for this year, I think.

You can see that all of these are for the love. I really dig on the Guile work that we've done the last year, and 2010 is the year to strut our stuff.

**non-nerd resoutions**

I guess the only one I really want to get out there is to be more real. Garden more. Chat more. Organize more. More human, more vegetable, less machine. How this meshes with my profession, or even my hack-desires, I have no idea.

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [closedgl particle simulation in guile](/archives/2013/02/16/opengl-particle-simulation-in-guile)
- [an in-depth look at the performance of guile's web server](/archives/2012/03/08/an-in-depth-look-at-the-performance-of-guiles-web-server)
- [the good hack](/archives/2009/07/26/the-good-hack)
- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)

### One response

1. [wingo](http://wingolog.org/) says:[27 January 2010 10:49 AM](#a0353e1e6317e4c91ffabf2889e32b92ea7691d4)

   I got a question from a friend here, asking if I was leaving Spain soon. The answer is no. In a couple of years, probably, yes; but I'm here for now. Thought I should clear that up :)

Comments are closed.
