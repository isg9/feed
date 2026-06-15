---
title: wikipedia & guile — wingolog
url: https://wingolog.org/archives/2009/10/01/wikipedia-guile
published: "2009-10-01T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/10/01/wikipedia-guile
---

# wikipedia & guile — wingolog

## [wikipedia & guile](/archives/2009/10/01/wikipedia-guile)

1 October 2009 3:49 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [wikipedia](/tags/wikipedia)
- [emacs](/tags/emacs)
- [gc](/tags/gc)

Good evening, internet!

Oftener then not, when posting dispatches to this ship's log, it's with an air of surprise: of the turned head over the shoulder, smiling at the unexpected appearance of a friend. Well hello world, then, it's just been like that, punctuated *ensimismamiento*.

\\* \\* \\*

This particular evening finds me in Los Angeles, here for work. I have loads of stories and photos, hopefully soon forthcoming, but the present impetus for text is more mundane. And nerdy, may I mention. For the laity, imagine a pendulous pocketwatch, you are getting sleepy, yes, yes -- you need not read the rest -- you might remember suddenly that you wanted to make some tea, and that upon your return to the machine will remember having read all of this log entry, and that its writer was a genteel and amiable fellow.

You should probably stop reading now, your tea water is aboil!

\\* \\* \\*

For the rest of you, please forgive the verbal excess, I've been reading too much Jane Austen recently.

My friend and colleague [B. Tregre](http://beepbeepboopboop.tumblr.com/) pointed me to Guile's [wikipedia page](http://en.wikipedia.org/wiki/GNU_Guile) today, basically asking, "is it true?"?

Because if you read that page, it's pretty clear that [Guile](http://www.gnu.org/software/guile/) is a bad thing. I mean, in deference to [Norvig](http://norvig.com/Gettysburg/), let's condense that page:

1. Guile

2. Implementation of Scheme

3. Since 1993

4. Embeddable as a C library

5. But let's get to the dirt

6. Scheme compliance

7. Still bitter about Guile following the 1998 Scheme standard but not the 1991 one

8. For some reason we're going to complain about Guile's call/cc implementation

9. Also we're going to demonize [conservative collectors](http://en.wikipedia.org/wiki/Garbage_collection_(computer_science)#Precise_vs._conservative_and_internal_pointers) some more.

10. Salt, wounds

11. Etc.

**meta**

Perhaps actually I should start with a meta-meta: I hack on [Guile](http://www.gnu.org/software/guile/) for a number of reasons. I like it, first of all. It's good software, widely deployed, with some [neat stuff](//wingolog.org/tags/guile/) in it. I like the folks that hack on it -- the people and what they do. I like the project's stated goals of freedom, and I like the hack. &c.

But what of that neat stuff shows up in the Wikipedia article? Practically nothing. Lack of enthusiasm in an encyclopedia is to be expected; but lack of information is something else.

The Guile wikipedia entry simply doesn't tell you much about Guile itself. So that's my first meta-complaint.

My second meta-complaint is perhaps more to the point -- that the *real* message, the message if you read between the lines, is "Guile is a bad implementation of Scheme". This is editorialization through tone and fact selection. That most of the facts are strictly true does not mean that the *message* is true.

So to me, while the facts of the article are mostly (though not entirely) right, the article itself is wrong. The selection of facts is not representative of Guile, the object under consideration.

**non-meta**

I guess it's worthwhile to discuss the principal argument against Guile expounded more explicitly in the article: namely, conservative garbage collection.

Conservative collection is a way to scan all memory, looking for pointers. If it finds something that looks like a pointer, it considers the pointed-to address to be in use. At the end of "marking" the memory, the collector "sweeps" all non-marked locations.

If you're confused but interested, I discussed this topic [last year](//wingolog.org/archives/2008/04/10/allocate-memory-part-one-of-n) in some detail. Wikipedia itself [has a good discussion](http://en.wikipedia.org/wiki/Garbage_collection_(computer_science)#Precise_vs._conservative_and_internal_pointers), too.

Guile now uses the [Boehm-Demers-Weiser collector](http://www.hpl.hp.com/personal/Hans_Boehm/gc/), instead of having an in-Guile implementation; Boehm himself [discusses the topic](http://www.hpl.hp.com/personal/Hans_Boehm/gc/issues.html) as well.

Basically, there is a potential problem with conservative GC: one might see an integer somewhere in memory, but mistakenly interpret it as a pointer, thus keeping around data unnecessarily. If this happened often enough, or on a big enough data structure, you could leak lots of memory.

In practice, this rarely happens.

In the very very unlikely event that you keep memory around that should be freed, there are tools to work around the condition; but in a language implementation, where you really control what's in addressable memory (both heap and stack), you might be able to eliminate such misidentifications altogether. Finally in a 64-bit system such a collision is highly unlikely. I've never had a problem with Guile leaking memory.

But, there are some programmers who see this hack (and it is a hack) as a Pavlovian bell for vitriol. I really don't get it. It's not like you have any options in plain C. And if your language implementation exposes itself broadly to C, your implementation doesn't have many options either.

PLT Scheme was able to [switch](http://www.cs.brown.edu/pipermail/plt-scheme/2006-June/013840.html) away from conservative collection because they were able to lessen their exposure to C, via development of a good in-Scheme foreign function interface (FFI). Guile will have this possibility in the future.

**culpability**

I've thought about these issues fairly hard. One of my goals is to get Guile into Emacs within the next 12-18 months. But what if pointers were consistently and persistently misidentified, making it impossible to keep Emacs sessions open for months or years? Then the snarky Wikipedians would add a paragraph explaining how it was Guile that broke Emacs.

So the question to consider is, what bit pattern is identified by libGC as a pointer, and what non-pointer words on your system have a value that matches that pattern? Pointers to heap-allocated objects are aligned on 8-byte boundaries, so their lower three bits will be 0. In addition, the pointer value must lie within an allocated heap segment, which typically are large values, at least a million or so. So integers that you typically see are most unlikely to collide.

Furthermore, while it is typically said that "certain integer values" can be mistakenly identified, those are integer values in C. Guile's representation of integers (at least, those less than 2^30 or 2^62) has the lower two bits of the integer to be "10" -- so a *Guile* integer will never be confused with a pointer.

What can be confused are native (C) integers. Native C integers can appear ephemerally on the stack, or in packed memory blocks, i.e. in a packed uint32 array. In the latter case, libGC would allocate that block as a "pointerless" segment, so the integers would not be scanned, leaving us only with the stack case.

For the stack, you have two cases -- code introduced by your runtime, e.g. the VM code, and code introduced by Scheme. I've already argued that Scheme values cannot be misidentified. The VM code is so small it is auditable -- and extremely unlikely to persistently store a large integer on the stack, because such integers are not used in the VM.

Even these stack unknowns will be eliminated when Guile does native compilation (within 12-24 months), so we completely control all bits on the stack, too.

That leaves the C runtime, which if there are problems in Emacs, they can be worked around with the libGC tools. But in the end, removing most GC annotations from Guile's runtime (and eventually, remove the GCPRO mess from Emacs) has hugely positive maintenance benefits -- and, I am convinced, no downsides in practice.

**stories**

I could conclude that "you shouldn't let others write your wikipedia pages" -- given a sufficiently broad conception of self-as-project -- but that makes Wikipedia look the victim here.

The ultimate meta-conclusion is more about message and transmission -- in short, about story. That is, to not let others define who or what you are. Make sure they are telling your story.

And if they aren't telling your story, to motivate your own story-tellers and retellers to get the word out. I'm hoping to get others to fix that article over the next few months, by getting guile-users subscribers to garden it a bit.

Until next time, intertube. Happy October!

## related articles

- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)
- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)

### 3 responses

1. Sam TH says:[1 October 2009 5:12 PM](#68b29548708b436009ce8080da4e58daeaa36918)

   That's not quite right about PLT Scheme. What enabled us to switch to a precise collector was the annotation techniques developed by Adam Wick \[1,2\], which transforms C code into new C code that cooperates with the collector.

   Also, while leaks weren't a huge problem for the CGC version of PLT Scheme, the new, precise collector is a significant performance win.

   \[1\] http://www.cs.utah.edu/~awick/

   \[2\] http://www.cs.utah.edu/plt/publications/ismm09-rwrf.pdf

2. [wingo](http://wingolog.org/) says:[1 October 2009 5:58 PM](#454c454efe632c1420b83c24aa21039ae619a377)

   Thanks for the corrections regarding PLT, Sam, and the pointers to more info. I'll check them out. Flatt does seem to suggest though, in that mail, that somehow PLT was hitting the conservative misidentification problem in practice.

3. Sam TH says:[1 October 2009 8:32 PM](#0f846f31182b39a00b615443d7f3af31ae8c10aa)

   Certainly, there were cases where CGC caused leaks, but I didn't find them to be a huge problem. Of course, I didn't run DrScheme for days at a time, either. I'm not a GC guy, but in hindsight I suspect that the ability to use modern GC technology, such as generational collection, is a bigger win than avoiding the leaks.

Comments are closed.
