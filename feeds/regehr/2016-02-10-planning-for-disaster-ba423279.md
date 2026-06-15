---
title: Planning for Disaster
url: https://blog.regehr.org/archives/1289
published: "2016-02-10T23:23:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=1289
---

# Planning for Disaster

Alan Perlis once said:

> I think that it’s extraordinarily important that we in computer science keep fun in computing. When it started out, it was an awful lot of fun. Of course, the paying customers got shafted every now and then, and after a while we began to take their complaints seriously. We began to feel as if we really were responsible for the successful, error-free perfect use of these machines. I don’t think we are.

This is a nice sentiment, perhaps even a defensible one if we interpret it as narrowly talking about academic computer science. On the other hand, probably within 20 years, there’s going to be a major disaster accompanied by the loss of many lives that is caused more or less directly by software defects. I don’t know what exactly will happen, but something related to transportation or SCADA seems likely. At that point we can expect things to hit the fan. I’m not optimistic that, as a group, computer scientists and computing professionals can prevent this disaster from happening: the economic forces driving automation and system integration are too strong. But of course we should try. We also need to think about what we’re going to do if, all of a sudden, a lot of people suddenly expect us to start producing computer systems that actually work, and perhaps hold us accountable when we fail to do so.

Obviously I don’t have the answers but here are a few thoughts.

- We know that it is possible to create safety-critical software that (largely) works. Generally this happens when the organizations creating the software are motivated not only by market forces, but also by significant regulatory pressure. Markets (and the humans that make them up) are not very good at analyzing low-probability risks. A huge amount of critical software is created by organizations that are subject to very little regulatory pressure.

- It is difficult to tell people that something is a bad idea when they very much want it to be a good idea. We should get used to doing this, following [Parnas’s courageous example](http://web.stanford.edu/class/cs99r/readings/parnas1.pdf).

- It is difficult to tell people that something is going to be slow and expensive to create, when they very much want it to be quick and cheap. We need to get used to saying that as well.

- We can and should take responsibility for our work. I was encouraged by the field’s generally very positive response to [The Moral Character of Cryptographic Work](http://web.cs.ucdavis.edu/~rogaway/papers/moral-fn.pdf). Computer scientists and computing professionals almost always think that their particular technology makes the world better or is at worst neutral — but that is clearly not always the case. Some of this [could be taught](http://randomwalker.info/publications/sw_engg_ethics.pdf).

- We need to be educating CS students in methods for creating software that works: testing, specification, code review, debugging, and formal methods. You’d think this is obvious but students routinely get a CS degree without having done any sort of serious software testing.

Finally, let’s keep in mind that [causes are tricky](http://web.mit.edu/2.75/resources/random/How%20Complex%20Systems%20Fail.pdf). A major human disaster usually has a complex network of causes, perhaps simply because any major disaster with a single cause would indicate that the system had been designed very poorly. [Atomic Accidents](https://blog.regehr.org/archives/1181) makes it clear that most nuclear accidents have been the result of an anti-serendipitous collection of poor system design, unhappy evolution and maintenance, and failure-enhancing responses by humans.

**Acknowledgments:** Pascal Cuoq and Kelly Heller gave some great feedback on drafts of this piece.
