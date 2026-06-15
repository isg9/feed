---
title: embracing conway's law — wingolog
url: https://wingolog.org/archives/2015/11/09/embracing-conways-law
published: "2015-11-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/11/09/embracing-conways-law
---

# embracing conway's law — wingolog

## [embracing conway's law](/archives/2015/11/09/embracing-conways-law)

9 November 2015 1:48 PM

- [conway's law](/tags/conway%27s%20law)
- [guile](/tags/guile)
- [hacking](/tags/hacking)
- [igalia](/tags/igalia)
- [bundling](/tags/bundling)

Most of you have heard of " [Conway's Law](http://melconway.com/Home/Conways_Law.html)", the pithy observation that the structure of things that people build reflects the social structure of the people that build them. The extent to which there is coordination or cohesion in a system as a whole reflects the extent to which there is coordination or cohesion among the people that make the system. Interfaces between components made by different groups of people are the most fragile pieces. This division goes down to the inner life of programs, too; inside it's all just code, but when a program starts to interface with the outside world we start to see contracts, guarantees, types, documentation, fixed programming or binary interfaces, and indeed faults as well: how many bug reports end up in an accusation that team A was not using team B's API properly?

If you haven't heard of Conway's law before, well, welcome to the club. Inneresting, innit? And so thought I until now; a neat observation with explanatory power. But as aspiring engineers we should look at ways of using these laws to build systems that take advantage of their properties.

**in praise of bundling**

Most software projects depend on other projects. Using Conway's law, we can restate this to say that most people depend on things built by other people. The [Chromium project](https://www.chromium.org/), for example, depends on many different libraries produced by many different groups of people. But instead of requiring the user to install each of these dependencies, or even requiring the developer that works on Chrome to have them available when building Chrome, Chromium goes a step further and just includes its dependencies in its source repository. (The mechanism by which it does this isn't a direct inclusion, but since it specifies the version of all dependencies and hosts all code on Google-controlled servers, it might as well be.)

Downstream packagers like Fedora [bemoan bundling](http://lwn.net/Articles/660429/), but they ignore the ways in which it can produce better software at lower cost.

One way bundling can improve software quality is by reducing the algorithmic complexity of product configurations, when expressed as a function of its code and of its dependencies. In Chromium, a project that bundles dependencies, the end product is guaranteed to work at all points in the development cycle because its dependency set is developed as a whole and thus uniquely specified. Any change to a dependency can be directly tested against the end product, and reverted if it causes regressions. This is only possible because dependencies have been pulled into the umbrella of "things the Chromium group is responsible for".

Some dependencies are automatically pulled into Chrome from their upstreams, like V8, and some aren't, like zlib. The difference is essentially social, not technical: the same organization controls V8 and Chrome and so can set the appropriate social expectations and even revert changes to upstream V8 as needed. Of course the goal of the project as a whole has technical components and technical considerations, but they can only be acted on to the extent they are socially reified: without a social organization of the zlib developers into the Chromium development team, Chromium has no business automatically importing zlib code, because the zlib developers aren't testing against Chromium when they make a release. Bundling zlib into Chromium lets the Chromium project buffer the technical artifacts of the zlib developers through the Chromium developers, thus transferring responsibility to Chromium developers as well.

Conway's law predicts that the interfaces between projects made by different groups of people are the gnarliest bits, and anyone that has ever had to maintain compatibility with a wide range of versions of upstream software has the scar tissue to prove it. The extent to which this pain is still present in Chromium is the extent to which Chromium, its dependencies, and the people that make them are not bound tightly enough. For example, making a change to V8 which results in a change to Blink unit tests is a three-step dance: first you commit a change to Blink giving Chromium a heads-up about new results being expected for the particular unit tests, then you commit your V8 change, then you commit a change to Blink marking the new test result as being the expected one. This process takes at least an hour of human interaction time, and about 4 hours of wall-clock time. This pain would go away if V8 were bundled directly into Chromium, as you could make the whole change at once.

**forking considered fantastic**

"Forking" sometimes gets a bad rap. Let's take the Chromium example again. Blink forked from WebKit a couple years ago, and things have been great in both projects since then. Before the split, the worst parts in WebKit were the abstraction layers that allowed Google and Apple to use the dependencies they wanted (V8 vs JSC, different process models models, some other things). These abstraction layers were the reified software artifacts of the social boundaries between Google and Apple engineers. Now that the social division is gone, the gnarly abstractions are gone too. Neither group of people has to consider whether the other will be OK with any particular change. This eliminates a heavy cognitive burden and allows both projects to move faster.

As a pedestrian counter-example, Guile uses the [libltdl](https://www.gnu.org/software/libtool/manual/html_node/Using-libltdl.html) library to abstract over the dynamic loaders of different operating systems. (Already you are probably detecting the Conway's law keywords: *uses*, *library*, *abstract*, *different*.) For years this library has done the wrong thing while trying to do the right thing, ignoring `.dylib`'s but loading `.so`'s on Mac (or vice versa, I can't remember), not being able to specify `soversion` s for dependencies, throwing a `stat` party every time you load a library because it grovels around for completely vestigial `.la` files, et cetera. We sent some patches some time ago but the upstream project is completely unmaintained; the patches haven't been accepted, users build with whatever they have on their systems, and though we could try to take over upstream it's a huge asynchronous burden for something that should be simple. There is a whole zoo of concepts we don't need here and Guile would have done better to include libltdl into its source tree, or even to have forgone libltdl and just written our own thing.

Though there are costs to maintaining your own copy of what started as someone else's work, people who yammer on against forks usually fail to recognize their benefits. I think they don't realize that for a project to be technically cohesive, it needs to be socially cohesive as well; anything else is magical thinking.

**not-invented-here-syndrome considered swell**

Likewise there is an undercurrent of smarmy holier-than-thou moralism in some parts of the programming world. These armchair hackers want you to believe that you are a bad person if you write something new instead of building on what has already been written by someone else. This too is magical thinking that comes from believing in the fictional existence of a first-person plural, that there is one "we" of "humanity" that is making linear progress towards the singularity. Garbage. Conway's law tells you that things made by different people will have different paces, goals, constraints, and idiosyncracies, and the impedance mismatch between you and them can be a real cost.

Sometimes these same armchair hackers will shake their heads and say "yeah, project *Y* had so much hubris and ignorance that they didn't want to bother understanding what *X* project does, and they went and implemented their own thing and made all their own mistakes." To which I say, so what? First of all, who are you to judge how other people spend their time? You're not in their shoes and it doesn't affect you, at least not in the way it affects them. An armchair hacker rarely understands the nature of value in an organization (commercial or no). People learn more when they write code than when they use it or even when they read it. When your product has a problem, where will you find the ability to fix it? Will you file a helpless bug report or will you be able to fix it directly? Assuming your software dependencies model some part of your domain, are you sure that their models are adequate for your purpose, with the minimum of useless abstraction? If the answer is "well, I'm sure they know what they're doing" then if your organization survives a few years you are certain to run into difficulties here.

One example. Some old-school Mozilla folks still gripe at Google having gone and created an entirely new JavaScript engine, back in 2008. This is incredibly naïve! Google derives immense value from having JS engine expertise in-house and not having to coordinate with anyone else. This control also gives them power to affect the kinds of JavaScript that gets written and what goes into the standard. They would not have this control if they decided to build on SpiderMonkey, and if they had built on SM, they would have forked by now.

As a much more minor, insignificant, first-person example, I am an OK compiler hacker now. I don't consider myself an expert but I do all right. I got here by making a bunch of mistakes in Guile's compiler. Of course it helps if you get up to speed using other projects like V8 or what-not, but building an organization's value via implementation shouldn't be discounted out-of-hand.

Another point is that when you build on someone else's work, especially if you plan on continuing to have a relationship with them, you are agreeing up-front to a communications tax. For programmers this cost is magnified by the degree to which asynchronous communication disrupts flow. This isn't to say that programmers can't or shouldn't communicate, of course, but it's a cost even in the best case, and a cost that can be avoided by building your own.

When you depend on a project made by a distinct group of people, you will also experience churn or lag drag, depending on whether the dependency changes faster or slower than your project. Depending on LLVM, for example, means devoting part of your team's resources to keeping up with the pace of LLVM development. On the other hand, depending on something more slow-moving can make it more difficult to work with upstream to ensure that the dependency actually suits your use case. Again, both of these drag costs are magnified by the asynchrony of communicating with people that probably don't share your goals.

Finally, for projects that aim to ship to end users, depending on people outside your organization exposes you to risk. When a security-sensitive bug is reported on some library that you use deep in your web stack, who is responsible for fixing it? If you are responsible for the security of a user-facing project, there are definite advantages for knowing who is on the hook for fixing your bug, and knowing that their priorities are your priorities. Though many free software people consider security to be an argument against bundling, I think the track record of consumer browsers like Chrome and Firefox is an argument in favor of giving power to the team that ships the product. (Of course browsers are terrifying security-sensitive piles of steaming C++! But that choice was made already. What I assert here is that they do well at getting security fixes out to users in a timely fashion.)

**to use a thing, join its people**

I'm not arguing that you as a software developer should never use code written by other people. That is silly and I would appreciate if commenters would refrain from this argument :)

Let's say you have looked at the costs and the benefits and you have decided to, say, build a browser on Chromium. Or re-use pieces of Chromium for your own ends. There are real costs to doing this, but those costs depend on your relationship with the people involved. *To minimize your costs, you must somehow join the community of people that make your dependency*. By joining yourself to the people that make your dependency, Conway's law predicts that the quality of your product as a whole will improve: there will be fewer abstraction layers as your needs are taken into account to a greater degree, your pace will align with the dependency's pace, and colleagues at Google will review for you because you are reviewing for them. In the case of Opera, for example, I know that they are deeply involved in Blink development, contributing significantly to important areas of the browser that are also used by Chromium. We at Igalia do this too; our most successful customers are those who are able to work the most closely with upstream.

On the other hand, if you don't become part of the community of people that makes something you depend on, don't be surprised when things break and you are left holding both pieces. How many times have you heard someone complain the "project A removed an API I was using"? Maybe upstream didn't know you were using it. Maybe they knew about it, but you were not a user group they cared about; to them, you had no skin in the game.

Foundations that govern software projects are an anti-pattern in many ways, but they are sometimes necessary, born from the need for mutually competing organizations to collaborate on a single project. Sometimes the answer for how to be able to depend on technical work from others is to codify your social relationship.

**hi haters**

One note before opening the comment flood: I know. [You can't control everything](https://twitter.com/noelwelsh/status/662319444849532929). You can't be responsible for everything. One way out of the mess is just to give up, cross your fingers, and hope for the best. Sure. Fine. But know that there is no magical first-person-plural; Conway's law will apply to you and the things you build. Know what you're actually getting when you depend on other peoples' work, and know what you are paying for it. One way or another, pay for it you must.

## related articles

- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)

### 12 responses

1. [Christopher Allan Webber](http://dustycloud.org/) says:[9 November 2015 3:19 PM](#da87979ab71a149b7cea5f738f130101f7cff23c)

   An interesting post! You've laid out a lot of the reasons why bundling becomes a tempting thing to do, and the problems that it ~solves... however, having been on both sides of the bundling game, I'm a bit worried that you're leaving a lot out.

   In MediaGoblin for a long time we used Python versions from the standard python packaging ecosystem and would resist pinning versions as much as possible. Things would break occasionally where someone would upgrade and everything would be out of sync, and for a time we tried pinning versions (and pinning versions is effectively very similar to bundling but with no modifications), which worked great for a bit, but then it turns out we never changed those pinned versions and all our library usage was massively behind... when we moved to upgrade, it was an even bigger nightmare than before. Worse yet, sometimes you find out that you've left something with security vulnerabilities pinned to an old version for a long time, endangering your users. Oops!

   On the Javascript side, we went straight-up with bundling, which worked for a long time, but we pretty much never updated them. It turns out once you've bundled something, the cognitive load to update them yourself ends up being extremely high, and far more complicated than if you had never done so in the first place. And also, yes, security vulnerabilities. And also, yes, we couldn't get packaged in Debian.

   So we tried unbundling the Javascript dependencies by introducing Bower! Well it turns out now you've added a package manager to your package manager so you can sob while you cry. I'm not sure this simplified our life much at all, both because now users have multiple language-side package managers that break at a time.

   Okay, I realize you didn't mention language package managers, but this turns out to be relevant to what you're saying because I got interested in packaging MediaGoblin for Guix, and I entered the whirlwind of npm's recursive packaging, and what a nightmare that is:

   http://dustycloud.org/blog/javascript-packaging-dystopia/

   So basically, here I am a program author, and I've spent more of my life fighting packaging and integration and blah blah issues than I would like to, by like, a factor of 100 or maybe 1000 or maybe something more hyperbolic.

   So you're right, bundling helped us a lot, but it introduced a whole bunch of other problems.

   Which leads me to wonder, this is a long post, and you have not even mentioned Guix. What do you think of its relative ability to help here? Part of the problem is that today, for most users, Guix does not really exist. Assuming it did though, imagine programs being able to be unpinned from specific versions until they had to be, and then they can pin themselves and not break the rest of the system! I think that's one of Guix's compelling features.

   (I'm not sure it'll always work for multiple packages that have conflicting package verisons in something that involves something like a $PYTHONPATH though... so maybe it's not a panacea for all languages, at least.)

   Interesting post, anyhow!

2. [wingo](https://wingolog.org/) says:[9 November 2015 4:13 PM](#f61d98c310f0ef87b3c00ac0dafa991dd5d21da4)

   Hi Christopher! I'm glad you mentioned Guix; I was thinking about that later. I guess I would observe that Guix-the-collection-of-packages is being developed by a socially cohesive group as a unified thing. I can identify the set of packages available in a system with a Git commit. To an extent Guix is an attempt to make real the fiction that there is one system, with the package objects acting as bundle-like buffers between the upstream developers and the Guix developers. In a sense the ambient authority that Guix-the-collection-of-packages eschews is the same thing as a source package with unspecified dependency versions, or a function with free variables.

   I like Guix for its bold attempt to build a practical, rational free software deliverable whose product is reproducible. If Guix pulled its packages instead from some random Github registry, it would be in exactly the same situation as NPM, for better \*and\* for worse!

3. [John Cowan](http://www.ccil.org/~cowan) says:[10 November 2015 1:32 AM](#a5cafa0042b90a3217dffd6ba8f072df2cc8c5a6)

   I talked about this once on LtU in a slightly different, more public context of server organizations that fix vexatious bugs only to break their clients. My example was a bug that causes stock price lookups for all organizations beginning with G to be too low by $100, or something like that. The trouble is that many clients have already worked around the bug, and the fix breaks them. What's the server to do? Both acting and not acting have bad consequences, just for different groups of people. (Versioning doesn't help, because many clients aren't bothering with the version, they just never noticed the results are gibberish before, and bumping the version will mean they won't get anything now.)

   We depend on other people's code because we want to inherit their work for free, but there is no "free". One way or another we pay for their mistakes. On the other hand, if we eliminate the dependency we must make (and pay for) our own mistakes. Which is worse is one of those highly path-dependent questions. In particular, most people cannot take Google's high-end approach such as you describe: if you are a small web company working in Python, like $EMPLOYER, you just have to take the good with the bad; you cannot afford to devote resources to the core Python dev team.

4. Sriram Ramkrishna says:[10 November 2015 1:48 AM](#4a2c7bb32d85c4004c642b4244fb3affac0bc338)

   This is a great post, and the part that really I felt was relevant to me was when you said:

   "On the other hand, if you don't become part of the community of people that makes something you depend on, don't be surprised when things break and you are left holding both pieces."

   When you do community management, it is always an opportunity when that does happen that you want people to invest in that upstream.

   I always feel irritated when people complain that GTK+ has become a GNOME project. That's because GNOME is the one investing in GTK+. If there was more projects who are using GTK+ and want to be involved in the development process then they should do that. Having more projects involved in GTK+ makes GTK+ a better all purpose toolkit that serves everyone's needs.

   Investing in your upstream community is critical at times. It's great that this is called out in Conway's Law.

5. [Arne Babenhauserheide](http://draketo.de) says:[10 November 2015 12:16 PM](#68a504108554e5ad624af29db909f01c4444083e)

   I’m missing something here: Why should I not fork Guile when I build a command line tool in Guile Scheme?

   Another point I’m unsure about, though related to this question, is that your points only seem valid to me if the problem at hand requires much less resources than you have available.

   The communication cost needs to be weighted against the cost of hiring and training more people.

   And I think the security argument only works because the money is not where the libraries are. It’s actually an economic argument: Compared to the shiny frontends, the core infrastructure (SSH, SSL, GnuPG, …) is almost unmaintained, and over long timespans a maintained project always wins against an unmaintained one in terms of security.

   Still becoming part of the community is important from a larger view: If downstream were invested in the community of upstream, the core infrastructure would not have become unmaintained in the first place. And despite my gripe with RedHat over systemd, their investment in the GCC community (and other parts of the free software community) is something I am deeply grateful for: It shows that they know that upstream pays part of their bills, so they have to invest into it, too.

6. Michael George says:[10 November 2015 6:29 PM](#f195f70cb38dabfbd19e77c59aa909585a61372d)

   In the end, it's a tradeoff. In my experience, people overestimate the risks of reuse and underestimate the risks (and costs) of developing something in-house.

7. [Ludovic Courtès](http://www.fdn.fr/~lcourtes) says:[11 November 2015 12:03 PM](#dd28fd5e853813a7732de828fca5655b3a6e8951)

   I clearly understand why an organization would want to bundle things. Taking the analogy with "Conway's law", the team that bundles another team's package is one that say: "I don't wanna talk to them."

   The free software community---by which I mean the group of freewill associated hackers, not large companies---is largely about talking to each other. Distros like Debian have been built as ways to create bridges among software teams. So it should come as no surprise that these very people who value the idea of working together dislike the idea of bundling---not to mention the actual, not fantasized, security issues that this creates.

   I'll have to disagree with the bits about libltdl, which are an opinion, not facts.

   But that's also because I feel a member of the Libtool community, even though I don't actually hack on it. I feel (hopefully rightly so!) that people working on it are friends, member of the same group. I don't remember stories about rejected patches.

   Speaking of being a member of a group: the place to discuss whether libltdl is appropriate in Guile is guile-devel; hint hint. ;-)

8. Micheal Dean says:[15 November 2015 0:12 AM](#905b5479383a2f1abfcd1ebdde856990f6aa7ac3)

   So anyone who might argue against the google's scheming of world domination is to be feared as a "hater", another one of the googlisms spread around by google's puppets.

   This "armchair hacker" has the Marxist's 6th sense, seeing naive "free software" for what it is. Free software cannot thrive in bourgeois society where labor is alienated from its product; the ruling class determines what software is acceptable, governed by the anarchy of the dis-functional world market.

   The Marxists have better social theory. Conway doesn't see beyond the given social order of the epoch.

   All I can say for Conway's "law" is that it mostly states the obvious trying to avoid Marxist conclusion.

   Slave software.

9. [daniels](http://www.fooishbar.org) says:[19 November 2015 11:37 AM](#96de292e054cbf53380c6c18f8f80b56f8a72209)

   Micheal, Andy is many, many things, but unaware of (and being afraid to discuss) the core tenets of Marxism is not one of them.

10. [Arne Babenhauserheide](http://draketo.de) says:[12 January 2016 9:26 AM](#570b935f77ce51e7c10d8326aa533ed593673ece)

    Regarding Bundling: Maybe this could be stated in a more refined way by distinguishing between projects which regularly break API and projects which are mostly stable.

    For a project which regularly breaks the API, staying up to date has a serious cost, so the net benefit of getting the latest development is much lower than for a project with a rock-stable API.

    (the background for this is the article on volatile software by Steve Losh: http://stevelosh.com/blog/2012/04/volatile-software/ — “breaking backwards compatibility” means “the program is broken”)

11. akohn says:[14 January 2016 10:36 PM](#3ab4be46c6bf491d6bf5476788c53565e51ccdcd)

    Failing to use existing components is either the necessary precondition for creating something new which offers (ideally) a reusable benefit, or else it is a waste of human life. We all lose when human life is wasted, so yeah, I care about it. Everyone who could benefit from advances in the technical commons should care about it, and anyone opposing in that regard needs to be contemned.

12. Robert Rogge says:[26 January 2016 1:19 PM](#6d8befd1d992302f810d984a7c793e4a063eb039)

    I wonder what applications Conway's law has outside of software development. If you move just outside of development to include users, or marketing objectives, or design objectives, or any other objectives, perhaps you can find other interesting ideas.

    I think that Conway's law might also explain how large bureaucracies produce "institutional-looking architecture" that imitates the pragmatic, machine-like nature of large human systems whose objective, while being essentially humane as in the case of schools and hospitals, is reduced to machine-like outputs that perhaps mimic the human-machinery of large state organizations.

Comments are closed.
