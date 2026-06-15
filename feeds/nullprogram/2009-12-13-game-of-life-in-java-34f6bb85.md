---
title: Game of Life in Java
url: https://nullprogram.com/blog/2009/12/13/
published: "2009-12-13T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/12/13/
---

# Game of Life in Java

## [Game of Life in Java](/blog/2009/12/13/)

December 13, 2009

nullprogram.com/blog/2009/12/13/

Sources:

```
git clone git://github.com/skeeto/GameOfLife.git

```

![](/img/gol/game-of-life.png)

Since I recently got back into Java recently, I threw together this little Game of Life implementation in Java. It looks a lot like my [maze generator/solver](/blog/2007/10/08/) on the inside, reflecting the way I think about these things. Gavin wrote a [competing version of the game](http://devrand.org/show_item.html?item=98&page=Project) in [Processing](http://processing.org/) which we were partially discussing the other night, so I made my own.

The individual cells are actually objects themselves, so you could inherit the abstract Cell class and drop in your own rules. I bet you could even write a Cell that does the [iterated prisoner's dilemma cellular automata](/blog/2007/11/06/). The Cell objects are wired together into a sort of mesh network. Instead of growing it wraps around on the sides.

It takes up to four arguments right now, with three types of cells, `basic`, implementing the basic Conway's Game of Life, `growth`, which is a cell that matures over time, and `random` which mixes both types together (seen in the screenshot). The arguments work as follows,

```
java -jar GoL.jar [ [  []]]

```

I may look into extending this to do some things beyond simple cellular automata.

- [java](/tags/java/)
- [interactive](/tags/interactive/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Game%20of%20Life%20in%20Java) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Game+of+Life+in+Java).

« [Tweaking Emacs for Ant and Java](/blog/2009/12/06/)

» [Magick Thumbnails](/blog/2009/12/21/)
