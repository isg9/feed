---
title: talks i would like to give in 2016 — wingolog
url: https://wingolog.org/archives/2016/01/21/talks-i-would-like-to-give-in-2016
published: "2016-01-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/01/21/talks-i-would-like-to-give-in-2016
---

# talks i would like to give in 2016 — wingolog

## [talks i would like to give in 2016](/archives/2016/01/21/talks-i-would-like-to-give-in-2016)

21 January 2016 11:59 AM

- [guile](/tags/guile)
- [luajit](/tags/luajit)
- [compilers](/tags/compilers)
- [talks](/tags/talks)
- [networking](/tags/networking)

Every year I feel like I'm trailing things in a way: I hear of an amazing conference with fab speakers, but only after the call for submissions had closed. Or I see an event with exactly the attendees I'd like to schmooze with, but I hadn't planned for it, and hey, maybe I could have even spoke there.

But it's a new year, so let's try some new things. Here's a few talks I would love to give this year.

**building languages on luajit**

Over the last year or two my colleagues and I have had good experiences compiling in, on, and under LuaJIT, and putting those results into production in high-speed routers. LuaJIT has some really interesting properties as a language substrate: it has a tracing JIT that can punch through abstractions, it has pretty great performance, and it has a couple of amazing escape hatches that let you reach down to the hardware in the form of the FFI and the DynASM assembly generator. There are some challenges too. I can tell you about them :)

**try guile for your next project!**

This would be a talk describing Guile, what it's like making programs with it, and the kind of performance you can expect out of it. If you're a practicing programmer who likes shipping small programs that work well, are fun to write, and run with pretty good performance, I think Guile can be a great option.

I don't get to do many Guile talks because hey, it's 20 years old, so we don't get the novelty effect. Still, I judge a programming language based on what you can do with it, and recent advances in the Guile implementation have expanded its scope significantly, allowing it to handle many problem sizes that it couldn't before. This talk will be a bit about the language, a bit about the implementation, and a bit about applications or problem domains.

**compiling with persistent data structures**

As part of Guile's recent compiler improvements, we switched to a [somewhat novel intermediate language](https://wingolog.org/archives/2015/07/27/cps-soup). It's continuation-passing-style, but based on persistent data structures. Programming with it is interesting and somewhat different than other intermediate languages, and so this would be a talk describing the language and what it's like to work in it. Definitely a talk for compiler people, by a compiler person :)

**a high-performance networking with luajit talk**

As I mentioned above, my colleagues and I at work have been building really interesting things based on LuaJIT. In particular, using the Snabb Switch networking toolkit has let us build an implementation of a "lightweight address family translation router" -- the internet-facing component of an IPv4-as-a-service architecture, built on an IPv6-only network. Our implementation *flies*.

It sounds a bit specialized, and it is, but this talk could go two ways.

One version of this talk could be for software people that aren't necessarily networking specialists, describing the domain and how with Snabb Switch, LuaJIT, compilers, and commodity x86 components, we are able to get results that compete well with offerings from traditional networking vendors. Building specialized routers and other network functions in software is an incredible opportunity for compiler folks.

The other version would be more for networking people. We'd explain the domain less and focus more on architecture and results, and look more ahead to challenges of 100Gb/s ports.

**let me know!**

I'll probably submit some of these to a few conferences, but if you run an event and would like me to come over and give one of these talks, I would be flattered :) Maybe that set of people is empty, but hey, it's worth a shot. Probably contact via the [twitters](https://twitter.com/andywingo) has the most likelihood of response.

There are some things you need to make sure are covered before reaching out, of course. It probably doesn't need repeating in 2016, but make sure that you have a proper [code of conduct](http://www.ashedryden.com/blog/codes-of-conduct-101-faq), and that that you'll be able to put in the time to train your event staff to create that safe space that your attendees need. Getting a diverse speaker line-up is important to me too; conferences full of white dudes like me are not only boring but also serve to perpetuate an industry full of white dudes. If you're reaching out, reach out to women and people of color too, and let me know that you're working on it. This [old JSConf EU post](http://2012.jsconf.eu/2012/09/17/beating-the-odds-how-we-got-25-percent-women-speakers.html) has some ideas too. Godspeed, and happy planning!

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [needed-bits optimizations in guile](/archives/2024/09/26/needed-bits-optimizations-in-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)

### 5 responses

1. [Sumana Harihareswara](http://harihareswara.net) says:[21 January 2016 4:14 PM](#8e71140f3afd24e254f69f3e4a8888b85a07d552)

   Remind me, do you already subscribe to Technically Speaking, which is pretty good IMO on reminding me of upcoming CfPs?

   http://www.techspeak.email/

   Best wishes and I hope you get just the right invitations and have a ball and learn a lot.

2. [wingo](Http://wingolog.org) says:[21 January 2016 8:56 PM](#9f3d7e356dba3bdaf2f3f0dd319f28adc5bc3ab1)

   I was not subscribed, but I am now. Thanks for the tip Sumana!

3. Matt Campbell says:[21 January 2016 9:56 PM](#82bb264c3d46499fcde5095d48e2d3b51f7e0d88)

   I would like to hear the "try Guile for your next project" talk.

4. [Emmanuel Oga](http://EmmanuelOga.com) says:[27 January 2016 8:36 AM](#d1ad65df2328d60dbd2cffb42b3c02295abbb1ec)

   My vote goes for building languages on luajit.

5. [Ulya](http://skvadrik.github.io/aleph_null/) says:[7 February 2016 9:41 PM](#f1f0200eece410423831256ba4d468eb05128cc3)

   I vote for compiler talk! Your compilation-related posts are really inspiring. Just the right type of reading when I'm tired and out of ideas. :)

Comments are closed.
