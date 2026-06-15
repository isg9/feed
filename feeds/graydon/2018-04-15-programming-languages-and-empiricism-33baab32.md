---
title: programming languages and empiricism
url: https://graydon2.dreamwidth.org/259333.html
published: "2018-04-15T06:06:05Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:259333
---

# programming languages and empiricism

Yesterday I had a somewhat incoherent conversation / argument about programming languages, over dinner with friends, and I wanted to write a more coherent and less argumentative version of here for future reference.

It's an argument that originates with a position I've seen stated in public discussions in the past: that language designers and implementers don't do enough empirical research, and that users and the platform-developers who motivate, fund and ultimately deploy languages make their decisions about what to build in an uninformed fashion, again due to a lack of empirical or objective data.

The metaphor used in the argument yesterday (which I've heard repeatedly before) was that of [evidence-based medicine](https://en.wikipedia.org/wiki/Evidence-based_medicine), with the analogy being that current language design, funding, and selection is being made the same way medicine was being practised before the era of evidence-based medicine (and [similar trends in other fields](https://en.wikipedia.org/wiki/Evidence-based_practice)). That is: we're doing our work strictly "by folklore and tradition", not "rigorous data" about how languages operate in the field.

I feel this metaphor (and argument) over-generalizes substantially, and both rests-on and reinforces a bunch of misunderstandings about what language design is and how languages are made. But it has *some* kernel of truth, and I want to acknowledge and state for the record that I'm no enemy of empiricism. I'd be happy to see more-rigorous empirical studies of the things *we have actual uncertainty about* and that *can be objectively quantified* from the field. More information -- and more scientifically-valid information -- can't hurt the design process, assuming it has an objective interpretation without tradeoffs (more on that below). But that's .. kinda where my agreement with the argument ends.

My disagreements with this metaphor and argument derive from the observation that as stated, it's mixing together several very different aspects of language-building (aspects that really don't translate to the medicine metaphor):

2. Primary engineering concerns: literally "what works" in a language.

3. Secondary engineering concerns: tradeoffs and weighting in design.

4. Quality of implementation: the labor that goes into each implementation.

5. Psycho-cognitive mental concerns: what's understandable, easy, fun, etc.

6. Socio-cultural ecosystem concerns: what "being a user" is like.

7. System-integration and infrastructure concerns: how it's situated.

I'm going to argue that language design includes at least all of these aspects -- is informed-by them, constrained-by them, considers them at length -- but that only some (say the third and fourth) are likely ripe for empirical research, and only the latter of those (cognitive concerns) could yield something individual practitioners can use, rather than organizations.

### Primary engineering: "what works"

Languages are at some level human engineering artifacts, consciously designed to achieve specific engineering goals. It is an ongoing field of research and development in which both entirely new approaches and different combinations of existing approaches are regularly built and evaluated. Typically new research is done at the scale of "toy languages" that are built to explore one or more concepts, to see if it "works" at a pilot-project level, and what is seen as promising at that scale is then scaled-up to an "industrial language" to see if [scale-dependent factors emerge](https://en.wikipedia.org/wiki/Pilot_plant#Scale_dependence_of_plant_properties) (and to investigate non-engineering concerns once the language is in the wild -- see later sections).

What "working" means in a "pure engineering" sense is usually relatively straightforward to evaluate: factors such as parsimony, expressivity, size and complexity of implementation, performance and various forms of scalability are easily (and regularly) measured and compared. While there can be (and is) significant disagreement about the composition and significance of certain benchmarks, the argument that no measurement or empiricism happens in the more obviously "engineering" aspects of languages is simply untrue: research papers, experience reports, internal evaluation at a project-management level, and external positioning by marketing and reviews regularly includes measurements of code sizes, human/machine automation ratios, error detection rates, compilation and execution speeds, etc.

### Secondary engineering: tradeoffs and weighting

So: certain engineering aspects of languages can be (and are) objectively evaluated. That does *not* mean that there is a single optimal setting on the design landscape of engineering concerns (even just considering the artifact and its immediate users in isolation). For two reasons:

2. Several engineering considerations are in somewhat-direct tradeoff with one another. We do our best to eliminate tradeoffs but when they exist one cannot optimize for both (or many!) simultaneously: improving (say) peak runtime performance or correctness may very well only be possible at the expense of (say) compilation speed or cognitive load.

3. Any given engineering consideration, even if objectively measurable and with an obvious "good" and "bad" orientation to its measurement, may have totally different *subjective value* to different audiences. For some people (say) online reconfiguration is more important than bounded latency; for others the ranking is the opposite. Both qualities can easily be evaluated (having either quality is obviously better than not) but the *relative value of each* is subjective.

When some improvement is made to some aspect of language engineering *not* in tradeoff with anything else it tends to be adopted quickly, across the board, and that tends to stick in subsequent languages. And to the extent to which differences in subjective value of objective differences exist, different languages often exist to cater to them separately.

To make a tired language analogy to cars: there's room in the market for both sports cars and pickup trucks, even while they both quietly adopt each year's improved brake pads and windshield wipers.

### QOI

It makes for boring "arguments on the internet", but language *implementations* are also just software artifacts: built by people in organizations, at a cost of time, and subject to management structures and software engineering practice. A great deal of what's good or bad about a "real" industrial language comes down to considerations of the people employed and underlying software-engineering process (and its various influences) that produced the implementation, not the on-paper design of the language.

While one *can* do empirical research here, it's not especially language-specific. It's research about how software projects manage risk, achieve quality, hire and manage skilled workers, estimate schedules and so on. Including how they gain adequate funding (given considerations of market power, maintenance costs, R&D grant-funding, government procurement requirements, etc.) and what their long-term strategic plans for the language are.

Empirical results here are also more likely to inform decisions made by the management chain of a language-development project, rather than language designers or implementers. The latter are usually not in a position to make such decisions, merely have to work in whatever employment circumstances they can find.

### Cognitive load of different features

Insofar as a language is a human/computer interface -- and I think this is a fairly important way of describing them -- it's meaningful to try to tune both sides of that interface to "fit" the needs of each party.

The machine side is *comparatively* easy to evaluate: machines are relatively narrow, deterministic things that we can (and do) run lots of experiments on, while doing engineering. As I said earlier, I don't think we're suffering from a lack of empiricism on that side.

The human side of the interface is a lot more challenging to evaluate, and if any part of the argument for "more empirical research" seems compelling to me, it's this: I'd like more numerical evidence about the relative *cognitive costs* of different (local) language features -- specific notations, abstractions or facilities -- and the limits of "ability to understand them" among various population(s) of users. Evaluation among practitioners here *is* driven a bit too much by folklore and rules of thumb, and additional clarity of "what's easy enough" or "what's too hard" for users would strictly improve our design choices.

That said, there are two big caveats to this desire:

2. Many features of a language are dictated by circumstances beyond simply "fitting within the cognitive budget", i.e. one of the other 5 aspects discussed in this argument simply requires the feature be present or absent, regardless of whether it's "cognitively affordable".

3. Many features can only be deployed in combination with, or exclusion of, other features, and/or have cognitive effects that are strongly conditioned on the presence of other features, i.e. they're impossible to test independently, in isolation.

### Human and cultural ecosystem around a language

Talking about languages -- especially when it comes to *choosing one to use* \-\- can often mean talking about much more than their on-paper designs *or* implementations: it often means talking about (and choosing between) different languages' *libraries* and *communities*. Even if you're an antisocial hermit who enjoys programming solo, working in a given language almost always entails contact with a substantial *literature* written in and about that language: its packages, documentation and "areas of focus".

This doesn't always happen by accident: often a language is designed specifically *for* an existing community of users, to address shortcomings of that community's current "standard language", or to anticipate the community's future needs. But often the development of a community and literature is very path-dependent, and cannot be isolated, predicted or reproduced: to "do Java again" or "do Perl again", for example, one would need to reproduce the entire sociological context of the early web. To "do Ada again" one would need to reproduce the context of the US military in the 1970s. Once a given culture starts to take shape around a language, feedback and self-selection often take control, cementing the language's "nature" (and even influencing its subsequent technical evolution) in ways that can't be deduced at all from the initial design. But that strongly influence who will want to use it, who it's appropriate for.

Such cultural developments happen at a given time and place, and while they're subject to analysis -- they *certainly* factor into choices people make about which language to use! -- they're about as challenging to do "rigorous empirical study" on (with hypotheses and experiments) as sociology or economics: you very occasionally get lucky and there's a somewhat-comparable minimal pair of scenarios to contrast, but usually any given pair of scenarios differs so many ways as to make conclusions very qualified, very uncertain.

### Technical situation of a language

What I just said about cultural context resisting empirical experiment goes double when talking about the technical situation of a language. This is perhaps the least-appealing, most disappointing, frustrating and honestly *compromised* aspect of language engineering, but also one of its most important: every language is born into, and must execute amid, a specific *technical context* of other languages, other libraries and other software systems, speaking other file formats and network protocols, running on other hardware and operating systems. These all have their own semantics, their own requirements, their own histories, their own *vast* investments of time and money and therefore technical *inertia*.

I've said this before, but to repeat myself: as much as Silicon Valley folks like to talk about this being a "young industry" it's really close to a hundred years old, and at least looking at the almost-50-years of technical infrastructure (re)built after the VLSI/microprocessor shift, many decisions are effectively set in stone at this point: changing them would cost tens or hundreds of billions of dollars, millions of person-years of effort. If you write a language -- especially one you wish to have a significant set of users -- you must be very aware of the technical situation it's going to exist within, and to interoperate with; and that technical situation will dictate a great deal of the design you pursue.

To talk about empirical research in this aspect of a language design, then, is even more wishful thinking than when talking about empirical research across communities of users: a language designer (or putative "empirical researcher") has *no meaningful way* to run a language-deployment experiment in a "different technical context" that doesn't include the historical development of the x86 ISA, or POSIX or Win32 filesystems or process semantics, or SQL RDBMSs, or TCP/IP sockets, or IEEE754 floats, or OpenGL, or Unicode, or all the standard and expected forms of linkage and representation, packaging and distrbution, diagnostics and debugging, clocks and coordination, interfaces and communication systems in a given technical environment. To the extent that language features are *dictated* by these considerations, they really are just a matter of puzzling out (and pricing) "how to be compatible", nothing the designer can experiment on, evaluate or change. One must simply learn how they work, and adapt one's design accordingly.

### Conclusion

Overall: I don't think it's meaningless to talk about increasing empirical research in ways that could benefit language design, but I don't think it applies to most of the things that influence a language's design or, more broadly, its trajectory in the world. Most of what decides a given language's fate is either well understood and universally adopted engineering practice; or well undersood and either niche-specific choices or tradeoffs; or depends on the money spent and developers hired by its sponsoring organization; or is path-dependent and unpredictable social factors; or is totally inflexible and dictated by technical context. The residual (mostly psychological) factors, sure: research away. But that's unlikely to dramatically alter the story of how languages get built, evolve, are chosen or deployed, persist or die off.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=259333) comments
