---
title: hospitalized for approaching perfection — wingolog
url: https://wingolog.org/archives/2009/06/08/hospitalized-for-approaching-perfection
published: "2009-06-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/06/08/hospitalized-for-approaching-perfection
---

# hospitalized for approaching perfection — wingolog

## [hospitalized for approaching perfection](/archives/2009/06/08/hospitalized-for-approaching-perfection)

8 June 2009 9:30 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [june](/tags/june)

My god, June, June must be the most amazing month of the year. Things are green, and the awareness is palpable that summertime is here, in the street, at the bars, at the beach -- this is the time that is not the other time.

June is when strange and wonderful things happen. My sister, who lives in DC, called the other day to say she'd be here in Barcelona this weekend. Just on Sunday I was walking to the park and ran into an old friend I hadn't seen since 2000. June is the month of picnics: of picnics on the Seine, of picnics in the park, of picnics on the beach, of barbecues on terraces. June is for bare feet and open windows. June is for fireworks and music in the street.

**hack**

And yet, I hack, so much. I'm growing a little concerned, to be honest -- but now more than ever I feel like I'm doing my best work, and so I continue. It might be an illness.

In case you're new here, I've been hacking on Guile, an implementation of Scheme. Since my last dispatch, I landed a branch to Guile master that brings psyntax to the heart of Guile, as the default expander.

This was actually quite a feat, given the [module hygiene work](//wingolog.org/archives/2009/05/10/the-very-merry-month-of-may) I wrote about earlier -- because how do you have an expander that understands modules and hygiene, before modules themselves have booted up? Anyway, it works now :)

For users, the practical implication of this is that syntax-rules and syntax-case are available, all the time. This is a lovely, lovely thing.

However, running the expander works against the most important performance hack of Guile 1.8: the memoizer. When you load a module, Guile 1.8 reads everything, but only does the minimum amount of work. For example, once it sees a procedure, it doesn't process the body of the procedure until the procedure is called. This reflects the reality that only a fraction of code is ever called, and was a big win for Guile 1.8.

But in Guile 1.9 / 2.0, we lose this advantage, as the expander does traverse all code. Of course, if your code is compiled already, it needs no expansion so loading is very very fast, but Guile has a lot of users and existing code out there, and I don't think they'd be particularly pleased if an upgraded touted to speed up their programs actually slowed them down.

So, for that reason, I hacked in some automatic compilation support, and turned it on by default. The upshot is that if Guile sees a file for which it can't find a corresponding compiled version, or the compiled version is out of date, it will compile it then and there -- and stash away the result, so it will be found the next time.

Of course, figuring out how to find the compiled files, and where to put the cache, but how to keep supporting shared installed architecture-specific compiled files, and how to cope with lack of permissions to write the compiled files, or errors in compilation -- you know, it's like Steve's [legalizing marijuana](http://steve-yegge.blogspot.com/2009/04/have-you-ever-legalized-marijuana.html). The devil's in the details. Hopefully I have things ironed out now, though. After all, if Python can do it...

Historically, expanding Guile Scheme with psyntax has had a couple more problems, though: the original variable names would be lost, due to the alpha renaming I discussed [before](//wingolog.org/archives/2009/05/10/the-very-merry-month-of-may); and, you lose source location information, so you don't know where backtraces come from. Through a stroke of luck and wine-induced hubris (thanks [Ángel](http://thomas.apestaart.org/log/?p=670)!), I did manage to plumb that information through, which is really pleasing.

(If you're still with me, I understand you are a programming languages enthusiast. Well to you I say: be wary of the verb "plumb". It suggests something shitty either about software of the process of making it. But I am willing to describe my interactions with psyntax as, indeed, shitty.)

(Continuing the parenthesis though -- are there not those that have a Freudian delight in shit? Whenever I hack psyntax I do feel better afterwards. It's a strange powerful relieved feeling.)

**...**

Perhaps the most relevant update from the last month in Guile is that we fixed a 2.0 release date, on the 15th of October; and we'll be releasing monthly until then, starting on the 19th of this month.

Also, Guile is finally joining the Unicode-speaking world, thanks to Ludovic Courtès and Mike Gran. We're using the little-known [libunistring](http://www.gnu.org/software/libunistring/) to do the heavy lifting; if there are packagers out in the audience, let this be my call to you to package this for your distros. It's by Bruno Haible, really well-done, and has been in Gnulib for a while now. It's time for it to go to distros.

Anyway, enough blathering for this Monday night. I was prompted into this by a fit of despair, looking again at the Waddell/Dybvig inlining algorithm. Perhaps after a more composed encounter with that paper, I'll have some novelties for the ether. Until then, happy hacking!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)

### 7 responses

1. [eoin](http://randomrules.org) says:[8 June 2009 10:49 PM](#d795382769e2c972804addc852461ca10f6fa766)

   It is indeed a wonderful, wonderful song.

2. [Grant Rettke](http://www.wisdomandwonder.com/) says:[8 June 2009 10:52 PM](#ea665e05576105a97c2ce4cfa657726ffeab4f30)

   How to handle automatic compiling and where to put the compiled files came up recently on the Ikarus discussion list:

   "Where should fasl files go?"

   and

   "Auto caching?"

   I would have posted the links but didn't want to get the post flagged:

   http://groups.google.com/group/ikarus-users

3. [Quim](http://flors.wordpress.com) says:[9 June 2009 5:26 AM](#ff3c5efe80c29ced6235389cb2c52e52830b7ce2)

   \> If you're still with me, I understand you are a programming languages enthusiast.

   Wrong! I'm just a friend of yours willing to meet you again in few weeks, captured by the "hospitalized" word in your blog subject. Man, you are using cheap TV soap opera techniques to increase your audience!

4. Abdulaziz Ghuloum says:[9 June 2009 6:29 AM](#0dddda6eb28660256c0ae34b8ae54412e34f18ed)

   So, I wrote this long response, clicked submit, and it told me "bad url" and ate my text. Bad blog!

5. Abdulaziz Ghuloum says:[9 June 2009 8:27 AM](#f0c44912a1b34f9ab6490e5ce22e330d6b0efb4b)

   Anyways, what I was saying is that the code for Waddell's inliner, or cp0, is available in Ikarus under ikarus/scheme/ikarus.compiler.source-optimizer.ss but the ikarus version does not implement some of the details like guaranteed inlining of singly-referenced vars. If you want to experiment with it, the easiest way is to do (new-cafe expand/optimize), type an expression, and see the result of expansion and optimization. The same trick works for Petite/Chez btw.

   Anyways, it looks like you're having good time bringing Guile on track.

   Aziz,,,

6. [wingo](http://wingolog.org) says:[9 June 2009 8:34 AM](#daa79b31abe19c7b0ecdc2e2c9a8ad9794f0e149)

   @eoin: Yes!

   @Grant: I do follow the ikarus list. Some of this was inspired by that discussion.

   My blog software doesn't flag anything, it just requires a valid subset of XHTML, and then gives rude error messages. Apologies for the latter bit :-/ The idea is that since git backs the whole thing, I can just revert comments if they're inappropriate, or revert the reversion if I made a mistake...

   @Quim: It wasn't intentionally dramatic ;) Just a reference to the lovely [Random Rules](http://wingolog.org/priv/random-rules.ogg). That album is \*fantastic\*.

   @Aziz: Thanks for the pointers, apologies for the comment filter, and thanks for the ongoing insight/inspiration on ikarus-users :)

7. [Dale Wiles](http://upcracky.blogspot.com/) says:[10 June 2009 3:07 AM](#6d78c39b6395ab974b396fc947a2c76b9d62942f)

   I caught this on Planet Gnome.

   I'm glad to see that Guile is alive and well. I'm really glad to hear it now supports Unicode.

   What's the current level of support for Gnome? I tried to check it out at www.gnu.org/software/guile-gnome but got a "page not found". Ominous!

Comments are closed.
