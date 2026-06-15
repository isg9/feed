---
title: A Modest Chess Engine
url: https://nullprogram.com/blog/2010/10/17/
published: "2010-10-17T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/10/17/
---

# A Modest Chess Engine

## [A Modest Chess Engine](/blog/2010/10/17/)

October 17, 2010

nullprogram.com/blog/2010/10/17/

```
git clone git://github.com/skeeto/october-chess-engine.git

```

![](/img/chess/chess-screenshot.png)

Here's what I've been working on for about the last week: a little chess application in Java. It has a built-in AI you can play against and it's capable of playing both standard chess and Gothic chess right now. It's actually a chess framework, so it would be fairly easy for me to add new chess variants, and the AI would be able to play it automatically. The hardest part is handling a particular chess variant's castling rule, since that's the hardest rule to generalize.

The AI is a minimax search over four full plies with a fairly simplistic heuristic (but more than just adding up material). It's probably not that great of a player, but it's good enough that none of my friends, nor I, are able to beat it. I concentrated more on making it flexible than fast, so it can't search more than five plies without taking unreasonably long. But it does use multiple threads to do the search, so it should be pretty speedy on an SMP system.

![](/img/chess/gothic-screenshot.png)

I thought about adding multiplayer capability and perhaps giving it the ability to interface with standalone chess engines (wow are those protocols ugly!), like [GNU
Chess](http://en.wikipedia.org/wiki/GNU_Chess). But there are other programs that already do this better than I ever could. I've done about as much as I was interested in doing with it.

Maybe you remember [that chess AI essay I
wrote some years back](/blog/2007/10/24/)? I was talking about using a genetic algorithm to tweak the AI parameters. Well, I actually tried to do that here with this AI (see the `OptimizeGA` class in the code). Turns out it's not very useful anyway! :-P It wasn't getting anywhere useful in my experiments.

- [java](/tags/java/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20A%20Modest%20Chess%20Engine) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=A+Modest+Chess+Engine).

« [Lorenz Chaotic Water Wheel Applet](/blog/2010/10/16/)

» [Java Applets Demo Page](/blog/2010/10/18/)
