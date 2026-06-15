---
title: 'research!rsc: Worms and Distributed Computations'
url: https://research.swtch.com/worm
published: "2008-02-20T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/worm
---

# research!rsc: Worms and Distributed Computations

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Worms and Distributed Computations Russ Cox February 20, 2008 *research.swtch.com/worm* Posted on Wednesday, February 20, 2008.

John Shoch and Jon Hupp's 1982 paper “ [**The ‘Worm’ Programs—Early**
**Experience with a Distributed Computation**](http://vx.netlux.org/lib/pdf/The%20%22Worm%22%20Programs%20-%20Early%20Experience%20with%20a%20Distributed%20Computation.pdf)” (PDF, also [HTML](http://vx.netlux.org/lib/ajm01.html)) discusses a fascinating series of distributed computation experiments carried out on a network of Altos at Xerox PARC. An [earlier post](http://research.swtch.com/2008/02/absolute-file-system-design.html) discussed the Alto file system.

The Alto had memory to run only one program at a time and lacked virtual memory. It multitasked by swapping one program (and often much of the operating system) completely to disk and then loading a different one. Such a “world swap” took about two seconds. Because of this usage model, users tended to close their programs before going home, leaving the machine idle for other uses at night. Because they came out at night, the programs were sometimes called “vampire programs.”

Shoch and Hupp called their programs “worms” after the electronic tapeworm created in John Brunner's 1975 book *[The Shockwave Rider](http://www.amazon.com/-/dp/0345248538/)*\*, although the distributed computation ideas probably predate the book. (They lament that the phrase “distributed computing” “has already been co-opted by those who market fairly ordinary terminal systems,” although, at least in my experience, the phrase seems to have been restored to its original meaning.)

The paper discusses a handful of possible distributed computations, ranging from a simple “hello,world” to a multimachine animation computation. Perhaps the most interesting part is Section 5, which gives some of the history of multimachine programs on the Arpanet (the predecessor of the Internet). Of course, there are the network's routers themselves; maintenance of routing tables is almost certainly the longest-running distributed computation in history. The paper gives a few other examples, most notably a multimachine air traffic control simulation called McRoss (multicomputer route oriented simulation system). McRoss partitioned the air space among the computers running the simulations; particular planes were handed off from one computer to another as they flew around the air space.

The summary concludes:

> This summary is probably not complete or fully accurate, but it is an impressive collection of distributed computations, produced within or on top of the Arpanet. Much of this work, however, was done in the early 70s; one participant recently commented, “It's hard for me to believe that this all happened seven years ago.” Since that time, we have not witnessed the anticipated blossoming of many distributed applications using the long-haul capabilities of the Arpanet.

In 2008, it's still hard to believe that this all happened 30 years ago. In the past decade, such distributed computations have really taken off ( [SETI@home](http://en.wikipedia.org/wiki/SETI@home) started in 1999). Today, [BOINC](http://en.wikipedia.org/wiki/Berkeley_Open_Infrastructure_for_Network_Computing) and [other projects](http://en.wikipedia.org/wiki/List_of_distributed_computing_projects) enlist the help of millions of computers to attack problems ranging from [chess](http://en.wikipedia.org/wiki/Chess960@Home) to [malaria](http://en.wikipedia.org/wiki/Malaria_Control_Project).

\\* The Shockwave Rider also contains this fantastically succinct introduction to the paradox of [orbital mechanics](http://dunningrb.wordpress.com/2007/10/12/fun-with-orbits-or-how-to-slow-down-by-speeding-up/):

> You are in circular orbit around a planet. You are being overtaken by another object, also in circular orbit, moving several km./sec. faster. You accelerate to try and catch up.
>
> See you later, accelerator.
>
> *Much* later.

The idea makes repeated appearances. Other wordings include “Go faster in order to drop back to a lower orbit? Doesn't work. Drop back to a lower orbit; you go faster!” and “The way to go faster is to slow down.”

(Comments originally posted via Blogger.)

- [Peteris Krumins](http://www.blogger.com/profile/15489446282405533799)(February 20, 2008 3:20 PM) Just wanted to let you know that your blog is great! Each day you post, I print out the article and read it during breaks between classes in university. Keep up the good work.

Sincerely,

Peteris Krumins

- Anonymous (January 4, 2011 7:28 AM) Couldnt agree more with that, very attractive article
