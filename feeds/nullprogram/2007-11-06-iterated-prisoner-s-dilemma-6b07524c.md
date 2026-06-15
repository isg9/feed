---
title: Iterated Prisoner's Dilemma
url: https://nullprogram.com/blog/2007/11/06/
published: "2007-11-06T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2007/11/06/
---

# Iterated Prisoner's Dilemma

## [Iterated Prisoner's Dilemma](/blog/2007/11/06/)

November 06, 2007

nullprogram.com/blog/2007/11/06/

![](/img/prison/top.gif)

I was reading about the [prisoner’s dilemma](http://en.wikipedia.org/wiki/Prisoner's_dilemma) game the other day and was inspired to simulate it myself. It would also be a good project to start learning Common Lisp. All of the source code is available in its original source file here:

- [/download/prison/prison.lisp](/download/prison/prison.lisp)

I have only tried this code in my favorite Common Lisp implementation, [CLISP](http://clisp.cons.org/), as well [CMUCL](http://www.cons.org/cmucl/).

In prisoner’s dilemma, two players acting as prisoners are given the option of cooperating with or betraying (defecting) the other player. Each player’s decision along with his opponents decision determines the length of his prison sentence. It is bad news for the cooperating player when the other player is defecting.

Prisoner’s dilemma becomes more interesting in the iterated version of the game, where the same two players play repeatedly. This allows players to “punish” each other for uncooperative play. Scoring generally works as so (higher is better),

Player AcoopdefectPlayer Bcoop(3,3)(0,5)defect(5,0)(1,1)

The most famous, and strongest individual strategy, is tit-for-tat. This player begins by playing cooperatively, then does whatever the its opponent did last. Here is the Common Lisp code to run a tit-for-tat strategy,

```
(defun tit-for-tat ()
  (lambda (x)
    (if (null x) :coop x)))

```

If you are unfamiliar with Common Lisp, the `lambda` part is returning an anonymous function that actually plays the tit-for-tat strategy. The `tit-for-tat` function generates a tit-for-tat player along with its own closure. The argument to the anonymous function supplies the opponent’s last move, which is one of the symbols `:coop` or `:defect`. In the case of the first move, `nil` is passed. These are some really simple strategies that ignore their arguments,

```
(defun rand-play ()
  (lambda (x)
    (declare (ignore x))
    (if (> (random 2) 0) :coop :defect)))

(defun switcher-coop ()
  (let ((last :coop))
    (lambda (x)
      (declare (ignore x))
      (if (eq last :coop)
          (setf last :defect)
          (setf last :coop)))))

(defun switcher-defect ()
  (let ((last :defect))
    (lambda (x)
      (declare (ignore x))
      (if (eq last :coop)
          (setf last :defect)
          (setf last :coop)))))

(defun always-coop ()
  (lambda (x)
    (declare (ignore x))
    :coop))

(defun always-defect ()
  (lambda (x)
    (declare (ignore x))
    :defect))

```

Patrick Grim did an interesting study about ten years ago on iterated prisoner’s dilemma involving competing strategies in a 2-dimensional area: [Undecidability in the Spatialized Prisoner’s Dilemma: Some
Philosophical Implications](http://www.sunysb.edu/philosophy/faculty/pgrim/SPATIALP.HTM). It is very interesting, but I really wanted to play around with some different configurations myself. So what I did was extend my iterated prisoner’s dilemma engine above to run over a 2-dimensional grid.

Grim’s idea was this: place different strategies in a 2-dimensional grid. Each strategy competes against its immediate neighbors. (The paper doesn’t specify which kind of neighbor, 4-connected or 8-connected, so I went with 4-connected.) The sum of these competitions are added up to make that cell’s final score. After scoring, each cell takes on the strategy of its highest neighbor, if any of its neighbors have a higher score than itself. Repeat.

The paper showed some interesting results, where the tit-for-tat strategy would sometimes dominate, and, in other cases, be quickly wiped out, depending on starting conditions. Here was my first real test of my simulation. Three strategies were placed randomly in a 50x50 grid: tit-for-tat, always-cooperate, and always-defect. This is the first twenty iterations. It stabilizes after 16 iterations.

```
(run-random-matrix 50 100 20 '(tit-for-tat always-coop always-defect))

```

![](/img/prison/random.gif)

White is always-cooperate, black is always-defect, and cyan is tit-for-tat. Notice how the always-defect quickly exploits the always-cooperate and dominates the first few iterations. However, as the always-cooperate resource becomes exhausted, the tit-for-tat cooperative strategy works together with itself, as well as the remaining always-cooperate, to eliminate the always-defect invaders, who have no one left to exploit. In the end, a few always-defect cells are left in equilibrium, feeding off of always-cooperate neighbors, who themselves have enough cooperating neighbors to hold their ground.

The effect can be seen more easily here. Around the outside is tit-for-tat, in the middle is always-cooperate, and a single always-defect cell is placed in the middle.

```
(run-matrix (create-three-box) 100 30)

```

![](/img/prison/boxes.gif)

The asymmetric pattern is due to the way that ties are broken.

The lisp code only spits out text, which isn’t very easy to follow whats going on. To generate these gifs, I first used this Octave script to convert the text into images. Just dump the lisp output to a text file and remove the hash table dump at the end. Then run this script on that file:

- [/download/prison/pd\_plot.m](/download/prison/pd_plot.m)

The text file input should look like this:

- [/download/prison/example.txt](/download/prison/example.txt)

You will need Octave-Forge.

The script will make PNGs. You can either change the script to make GIFs (didn’t try this myself), or use something like [ImageMagick](http://www.imagemagick.org/) to convert the images afterward. Then, you compile frames into the animated GIF using [Gifsicle](http://www.lcdf.org/gifsicle/).

See if you can come up with some different strategies and make some special patterns for them. You may be able to observe some interesting interactions. The image at the beginning of the article uses all of the listed strategies in a random matrix.

I will continue to try out some more to see if I can find something particularly interesting.

- [ai](/tags/ai/)
- [video](/tags/video/)
- [lisp](/tags/lisp/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Iterated%20Prisoner's%20Dilemma) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Iterated+Prisoner%27s+Dilemma).

« [Chess AI Idea](/blog/2007/10/24/)

» [Neural Network Blackjack Game](/blog/2007/11/13/)
