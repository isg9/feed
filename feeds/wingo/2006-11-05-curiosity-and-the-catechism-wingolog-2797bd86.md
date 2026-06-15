---
title: curiosity and the catechism — wingolog
url: https://wingolog.org/archives/2006/11/05/curiosity-and-the-catechism
published: "2006-11-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/11/05/curiosity-and-the-catechism
---

# curiosity and the catechism — wingolog

## [curiosity and the catechism](/archives/2006/11/05/curiosity-and-the-catechism)

6 November 2006 2:11 AM

- [scheme](/tags/scheme)
- [america](/tags/america)
- [spain](/tags/spain)
- [version control](/tags/version%20control)
- [arch](/tags/arch)
- [writing](/tags/writing)
- [harper's](/tags/harper%27s)
- [guile](/tags/guile)

I have a bit of a writing backlog. Rather than edit edit edit, taking the moment out of whatever it was I was writing, I'm just going to dump a bit before writing something new.

**new books**

I would like to make words about John Leonard.

I recently stole half a dozen back issues of Harper's from a friend's apartment in the states. It is my purloined word-horde of delight.

Their happiest turns of phrase are offered by Leonard's monthly book reviews. I imagine him as an eccentric spider in a multidimensional web, nimbly turning around newly trapped books in webs of their predecessors, decorating his subjects with perception. Generous, too, like a grandfather in his shop, talking out loud, telling old stories. Then he give you his tools, asks you to try your hand at the lathe or soldering iron.

(My grandfather was a spider, it make my eyes mist thinking of him, excuse me)

**spain**

I'm growing a bit frustrated with Spain, on this my fourth anniversary of flying away from my previous homes in the states. Why can't I find tofu or decent sliced bread in the stores? Why is it that my schedule overlap with grocery stores is only 40 minutes per day? Why is it so difficult to find a café to hack in at three in the afternoon on a sunday? Not to mention the lack of greasy spoons, burritos, and proper sandwiches.

Say what you will about cultural relativity, but a grilled emmenthal-walnut-basil-avocado-mustard sandwich on hearty dark bread is objectively better than flaccid bacon and processed cheese on a dry baguette.

On the other side of the exaggeration, Barcelona is civilized in ways that American towns don't even know how to dream about. I don't miss the irritations of owning a car. I can bike everywhere in town. When I go out my front door, there are people walking the streets, strolling with and without purpose -- the liquid to the gaseous state of America. What is not here is the second-hand couch on the front porch, the rocking chair, the shed out back.

Apparently my happiness is entirely determined by home, food, and transportation. I fret insatiable.

**hacking**

Hacking, I take control of my life. Or perhaps the clause should be, "doing things I should have done a while back". I realized this the other day that after knocking down some bugs in [guile-gnome](http://www.gnu.org/software/guile-gnome/), [guile-lib](http://home.gna.org/guile-lib/), and [g-wrap](http://www.nongnu.org/g-wrap/). Doing so lets me take care of email backlogs of bug reports, patches and questions I never got around to before, making me feel like I'm actually getting on top of my inbox, which is currently at best a minor form of guilt.

Speaking of guilt, I should mention something that people nagged me about for a long time, until apparently they gave up: the video archives for GUADEC 2006. Here is the situation. The raw recordings I have are of large chunks at a time (between 2 and 20 hours), and do not play properly in most players. You cannot seek in them. Why? Because I fucked up and recorded in too high a quality for the boxes we had for encoding. Secondarily, after we had to drop some frames, the encoders continued on as if frames had not been dropped, thereby ensuring that the archives have large synchronization problems.

At first I invested quite a bit of effort into trying to get these videos cut and resynchronized, writing two applications and a few hacks to a number of GStreamer elements. Last time I looked, those hacks were not working properly (segfaults, etc). It was depressing on the three levels of (1) I fucked up in the beginning, (2) I wrote large parts of the capturing software, and I didn't think it would discard the timestamps, and (3) the attempts at getting out ok-to-decent archives were failing also due to code I needed to write.

Given a limited amount of personal hack time, I chose to hack on my guile-related projects. Much more personal bang for the buck. While guilt might be useful on some occasions, and is only a two-key typo away from guile, in this case it was too much and I had to back off to retain my sanity.

So, um, my bad about that guys! I know it sucks. I've got some folks at the office interested in getting this job done so hopefully before the end of the year some decent archives will be out.

**good work**

(Is there some kind of official body or church or something that one can go to for egoism problems? Looking over this next paragraph, I seem to need it.)

Going over the guile stuff I did, I have to say there is some really good work there. It's what continues to attract me to those projects. The [texinfo parser](http://localhost/~wingo/ambient/software/guile-lib/texinfo/) I wrote for guile-lib is pretty hot, even given my proclivity for parsers. The lazy bindings work I did for g-wrap was all right. Mapping a GTK text entry to a scheme port wasn't so bad either. I dig on hacking it.

Distributed version control systems promote bitrot. With centralized systems, either your code is in or it's not: if you want it in, you have to get it in the maintained trunk. With decentralized systems, you can commit your code to some branch somewhere, and mentally mark it as done. This week I found patches *over two years old* lingering in one of my arch branches of guile-lib. Two years. People had been writing in to mailing lists to complain about it, and I was wondering why they didn't have the fixed version. Sheesh.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [random things](/archives/2007/11/08/random-things)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)

### 8 responses

1. Peter Lund says:[6 November 2006 2:38 AM](#217)

   So, not only is the perfect the enemy of the good, the good can also be the enemy of better than nothing ;)

   (regarding the GUADEC videos)

2. [Luis](http://tieguy.org/) says:[6 November 2006 3:17 AM](#218)

   European sandwiches in general are just pale, pale imitations of a great American sandwich. But maybe there is a market there...

3. [Luis](http://tieguy.org/) says:[6 November 2006 3:21 AM](#219)

   (by market I mean 'way to subsidize living in Barcelona' ;)

4. Juanjo Marin says:[6 November 2006 4:26 AM](#220)

   "Why can’t I find tofu or decent sliced bread in the stores? Why is it that my schedule overlap with grocery stores is only 40 minutes per day? Why is it so difficult to find a café to hack in at three in the afternoon on a sunday? Not to mention the lack of greasy spoons, burritos, and proper sandwiches."

   ...Maybe 'cause the cultural gap :)

   Relax yourself and enjoy the difference !!!!

   Anyway, It's hard to believe that you can't find tofu in Barcelona. I live in a small south town in Spain and I can buy tofu on 3 shops, so.......

5. [Janne](http://www.lucs.lu.se/people/jan.moren/log/current.html) says:[6 November 2006 5:28 AM](#221)

   "decent sliced bread" - I think you can find a clue in the combination of those three words. How about buying fresh, non-sliced (gasp!) bread and, you know, slice it yourself?

   As for sandwiches, I can't speak for the sandwich situation in central Spain, but in even small towns in Sweden you can find any number of variations of very, very good, fresh sandwiches, bagle, rye or based on various italian breads. True, what they aren't is unappetizingly oversized, but if you're that hungry, just buy two.

6. [wingo](http://wingolog.org/) says:[6 November 2006 2:21 PM](#222)

   Janne. A decent unsliced loaf would admittedly be delicious. Then afterwards I would take a slice of it and make a delicious sandwich.

   I am not referring to *pan de molde*!

7. [Ricky Youngblood](http://www.macgregor26.com/) says:[6 November 2006 11:39 PM](#223)

   One of my great joys in Barcelona was eating a avocado, tomato, and olive oil soaked sandwiches on a fresh bread for lunch while polishing off a bottle of wine. Also, I had no problem finding tofu. There was a healthfood store 5 blocks from where I lived.

   "Barcelona is civilized in ways that American towns don’t even know how to dream about..."

   Ahem. Except for San Francisco. We have cabins too... Strangely enough the sandwich situation in my part of town is not good. I wonder if there is something going on with that...

8. Ludovic says:[7 November 2006 0:18 AM](#224)

   Hi.

   Did you had a shot at Corte Ingles supermaket ? My mexican and italian flatmate seems to find whatever they want there.

   That said, I have to admit that I miss bread diversity too and creamy colorful french cakes. Here there are rather dry :)

Comments are closed.
