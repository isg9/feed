---
title: What&#8217;s Operating Systems Research About?
url: https://blog.regehr.org/archives/798
published: "2012-09-14T18:54:16Z"
feed: regehr
guid: http://blog.regehr.org/?p=798
---

# What&#8217;s Operating Systems Research About?

![](http://upload.wikimedia.org/wikipedia/commons/2/2f/Priv_rings.svg)The other day at lunch I tried to explain to [Suresh](http://geomblog.blogspot.com/) what operating systems research is all about, which got me thinking about this subject. As a quick glace at the [OSDI 2012 program](https://www.usenix.org/conference/osdi12/tech-schedule/osdi-12-program) will confirm, the obvious answer “it’s about building operating systems” no longer applies, if it ever did. In fact, the trend away from working mainly on ring 0 code was noted more than 12 years ago in [Rob Pike’s entertaining screed](http://herpolhode.com/rob/utah2000.pdf) (which I regrettably missed—he visited Utah just a few months before I got here). Pike said “the situation is genuinely bad and requires action” but it’s clear that most of his observations are simply symptoms of a maturing field. For example, iOS and Android are systems that I would consider innovative, but they are respectively based on BSD and Linux kernels that Pike would consider completely boring. The fact is: these kernels work. Significant kernel innovation was not required to make modern tablets and smart phones viable. If we take a strict definition of operating systems research, a lot of the interesting work since 2000 has been in virtualization. In fact, it is curious that Pike’s slide deck does not mention virtualization since the modern wave of hypervisors (which originated in academia) was well underway by the time he gave his talk.

So there exists this moderately large community (the last SOSP I attended, in 2009, had 565 attendees) of bright people who understand systems, who aren’t afraid to get their hands dirty, and who want results instead of theorems. But also, the bar for creating a new OS has gotten higher and higher for reasons that Pike describes (there’s a lot of hardware to support and a lot of standards to implement) and also for an important reason that he fails to mention (operating systems already work pretty well). Does this community disband? No, they stick together but the kinds of problems being addressed become more diverse, as the OSDI program illustrates nicely.

It would be sad if a community existed only due to inertia, but that is not the case here. I would claim that the thing holding the OS community (as it exists today) together is a common approach to doing research. I’ll try to characterize it:

1. The best argument is a working system. The more code, and the more results, the better. Something that is clearly a toy isn’t convincing. It is not necessary to build an abstract model, conduct a user study, prove soundness, prove correctness, or show any kind of asymptotic bound. In fact, if you want to do these things it may be better to do them elsewhere.
2. The style of exposition is simple and direct; this follows from the previous point. I have seen cases where a paper from the OS community and a paper from the programming languages community are describing almost exactly the same thing (probably a static analyzer) but the former paper is super clear whereas the latter is incredibly difficult to figure out. To some extent I’m just revealing my own biases, but I also believe the direct approach to exposition is objectively better; I’ll try to return to this subject in a later post.
3. The key to a strong research result is finding the right abstraction. A good abstraction is beautiful; it imposes little performance penalty; it leads to reliable systems; it leaks the right information and blocks things you didn’t want to know. It just feels right. The abstraction is probably for something low-level, but this doesn’t need to be the case. Finding good abstractions may sound easy but it’s super hard, often requiring lots of code to be thrown away multiple times.

And that, friends, is what OS research is about.

**UPDATE from 9/15:**

In a comment, Suresh says:

> It seems to me that you need to be able to list core problems that you want to solve, or things you want to understand. OS as “the study of interfaces” seems overly broad, and characterizes really any system building effort, even if it’s in databases or in a public-key infrastructure.

At the level of an entire subfield I’m not sure you can construct a satisfying list of core problems to be solved. What would this be for software engineering? It would be something extremely vague like “enable predictable, low-cost creation of acceptable software.” Yuck. How about for programming languages? For scientific computing? For theoretical computer science? Obviously we can come up with something, but I think that at this level the approach matters more than the specific problems. The problems tend to come and go over a period of a few years. Some of them (e.g. efficient virtual memory, efficient hypervisors) get solved while others (concurrent programming, secure operating systems) end up being harder than we thought and slowly morph into more tractable versions.

Anyway, I’m bummed that you find this unsatisfying but it’s the best I can do right now. Maybe someone else can do better.

Bhaskar states that “theorems are a kind of result” and of course I agree. However, they are not a kind of result often seen in OS research, which is the only thing I was trying to talk about. He also says:

> You first note that the simple and direct style of exposition for systems papers follows from the absence of a need to prove anything rigorous, and then indicate that this style is objectively better. This sounds contradictory to me. The more descriptive style preferred by systems researchers indeed follows from the fact that their core “rigorous/formal” component is code, which is not part of the paper itself; for communities where that component is, say, a proof, it must be presented within the paper itself. The paper consequently needs to be written with more precision, and may consequently be harder to read, particularly to those outside the field. As with any form of literature, the rhetorical style must match the purpose.

First, I like the bit about “their core rigorous/formal component is code” — that’s a great way of putting it.

Second, I shouldn’t have said that the writing style follows from the lack of proofs. Of course there exists wonderfully clear mathematical writing. However, the “OS writing style” does benefit from a relatively baggage-free research framework in which the world is its own best model.

Finally we come to the fun part: “as with any form of literature, the rhetorical style must match the purpose.” Of course this is true, but here we have been given this great gift where through some process of convergent evolution, researchers from different communities have ended up not only attacking the same problem, but also coming up with very similar solutions. If we look at some specific kinds of static analysis and bug finding, we can find papers from software engineering, from formal methods, from programming languages, and from operating systems that are doing essentially the same thing. Thus the purposes are the same. Even so, the rhetorical styles are very different. So we have form not following function, but rather following tradition. I’ve seen this happen a number of times. Reading these papers back to back is kind of like watching *Rashomon.*
