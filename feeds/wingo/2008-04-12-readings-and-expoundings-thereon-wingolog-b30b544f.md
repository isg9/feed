---
title: readings, and expoundings thereon — wingolog
url: https://wingolog.org/archives/2008/04/12/readings-and-expoundings-thereon
published: "2008-04-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/12/readings-and-expoundings-thereon
---

# readings, and expoundings thereon — wingolog

## [readings, and expoundings thereon](/archives/2008/04/12/readings-and-expoundings-thereon)

12 April 2008 11:11 PM

- [mako](/tags/mako)
- [martin](/tags/martin)
- [democracy](/tags/democracy)
- [anarchism](/tags/anarchism)
- [smalltalk](/tags/smalltalk)
- [alan kay](/tags/alan%20kay)
- [lambda the ultimate](/tags/lambda%20the%20ultimate)

**the reading**

Alan Kay's [The Early History of Smalltalk](http://gagne.homedns.org/~tgagne/contrib/EarlyHistoryST.html).

> New ideas go through stages of acceptance, both from within and without. From within, the sequence moves from "barely seeing" a pattern several times, then noting it but not perceiving its "cosmic" significance, then using it operationally in several areas, then comes a "grand rotation" in which the pattern becomes the center of a new way of thinking, and finally, it turns into the same kind of inflexible religion that it originally broke away from.

**the expounding**

The most interesting individual I ran into at last month's [Campaign for Real Ale and Common Lisp](http://cracl.wordpress.com/) meeting in LA was a Smalltalk hacker. I was showing off [tekuti](//wingolog.org/software/tekuti/), boasting about how you could hack it from Emacs while it's running via [GDS, the Guile debugging system](http://article.gmane.org/gmane.lisp.guile.user/6423), and the fellow scoffed, his beard moving up and down in laughter.

Smalltalk is a language, yes, he explained, but it's also a system: an integrated environment that you hack from the inside. The editor is not separated from the program via a process boundary: the objects of the program are immediately available to tinker with. So, for example, [SLIME](http://common-lisp.net/project/slime/) is but a pale echo of what is available to a Smalltalk hacker.

I admit I don't fully get this -- how do processes and threads enter into the picture? How can so much imperative state-modification scale to programming in the large? But what I do know is that especially in the young and naive field of computer science (ooh, a widget!), we forget more than we know. SLIME (and Guile's GDS) undoubtably does form a kind of message-passing, but it is necessarily limited, having been bolted on, not existing as a native linguistic construct.

Already now, I am unable to fully use the CPUs that I have at work. The programs I run are too primitive to make use of all of its cores. Processing capability is fast outracing the Von Neumann bottleneck, creating a "market" for different kinds of programs, ones that run in heterogenous, asynchronous environments -- distributed computing and NUMA and all of that.

Something new has to come, and the new thing will be built on the best strains of the past: ideas that cut with the grain of multicentered programming. These ideas have nothing to do with C++ or with pthreads.

This would be the point at which I should say that we have to mine the past to create a new future, but I am a [lisp weenie](http://smuglispweeny.blogspot.com/2008/02/all-your-parentheses-are-belong-to-us.html), and so I hold that the past *is* the future, in some form. Also, regarding Smalltalk, [Lambda, the ultimate political party](http://www.nhplace.com/kent/PS/Lambda.html): we lispers are too afraid of smalltalkers to call them our own. So it appears that the present will again slump into a future of an uneasy détente.

**the reading**

[The Geek Shall Inherit the Earth: My Story of Unlearning](http://mako.cc/writing/unlearningstory/StoryOfUnlearing.html), by Benjamin Mako Hill.

> Before \[taking a college course called "Cyberlaw"\], I had only considered free software and my involvement with free software advocacy and development as a tool I might use to accomplish other goals and activities. Before this point I saw myself as writer, a programmer, or a computer scientist. At this point, I was exposed to people who were, first and foremost, activists, idealists, and advocates of a system that I understood. After this point, I allowed myself to be seen--by others and by myself--as an activist and a radical, first and foremost--on my terms and in a way that I felt completely comfortable with.

**the expounding**

Mako writes about his personal history, a story of his journey to and then with free software, and then to a larger conception of what it means to be a human being. His holistic approach to existence refreshes the soul. There are ties between what we do in the day, what we do in the night, and what happens in the world: to have that much- [dream'd](http://classiclit.about.com/library/bl-etexts/wwhitman/bl-ww-idream.htm) new world, we must change the practice of our days.

Also, an afterthought. As time goes by, I slip farther down Google's "wingo" search results. I was convinced this was a sign of the expanding internet, or a castigation for egotism, but today I was consoled a bit, seeing [Mako](http://mako.cc/copyrighteous/), a much, much better known person to the internets, out of the top five for his name.

While I am not in a position to judge myself, certainly in Mako's case this is not a good thing: Mako's writings are much more interesting and important than the price of whatever stock is denoted by the MAKO ticker.

**the reading**

[Democracy without elections](http://www.uow.edu.au/arts/sts/bmartin/pubs/95sa.html), by Brian Martin (1995).

> It should be a truism that elections empower the politicians and not the voters. Yet many social movements continually are drawn into electoral politics.
>
> \[Elections operate\] to bring mass political activity into a manageable form: election campaigns and voting. People learn that they can participate: they are not totally excluded. They also learn the limits of participation. Voting occurs only occasionally, at times fixed by governments. Voting serves only to select leaders, not to directly decide policy. Finally, voting doesn't take passion into account: the vote of the indifferent or ill-informed voter counts just the same as that of the concerned and knowledgeable voter. Voting thus serves to tame political participation, making it a routine process that avoids mass uprisings. The expansion of suffrage helps to reduce the chance that a revolt by an oppressed or excluded group will be seen as justified; with the vote, it is easy for others to claim that they should have used 'orthodox channels.'

**the expounding**

The baldfaced fascism of the current American regime produces in many a sense of nostalgia for the benign Clinton regime, the one in which "the others" were consigned to opposition, and "we" controlled the presidency. The daily fire-and-motion given to us by the American news media machine, discussing the topics given to them by White House spokesmen, in White House terminology, makes the intellect so absorbed in *reaction* that constructive *production* is difficult, and impossible for many.

But let us not forget that even the conservative UNICEF estimates that 500,000 Iraqis, mostly children, died as a result of the Clinton-imposed Iraq sanctions. The real number is assuredly higher, and only with the most assiduous efforts is the Bush administration finally able to contemplate surpassing this number.

This is Genocide, by "both" parties.

Electing a "Democrat", of whatever color or gender, will not change this. No progressive change has ever resulted from the election of a so-called leftist politician: changes are the result of the grassroots action of organized people. Changes in the figureheads are a mere symptom.

I used to have the "unsophisticated" viewpoint described by Martin, in which I viewed both parties as fundamentally serving the cause of institutionalized violence, and therefore equal. Well while I might believe the former, the latter is not the case: strategic voting has effects. See for example the Communist party's abstention in the German elections in '33 or so, in which they considered the National Socialists to be equal to the Christian Democrats, and thus abstained with tragic results. The truth is that you always have to have your values with you, to see the choices that are available -- and, like Mako says, often they are not the choices given -- and to make your choices according to your values.

**the listening**

I *adore* Sonic Youth's "Washing Machine".

## related articles

- [but that would be anarchy!](/archives/2013/06/13/but-that-would-be-anarchy)
- [sindicat, dia zero](/archives/2010/02/13/sindicat-dia-zero)

### 6 responses

1. [Chris Cunningham](http://blondechris.com) says:[13 April 2008 10:45 AM](#febddc10ee9e7d031d082b6d69e4390f015da382)

   I'm interested in your explanation of Franklin Delano Roosevelt's presidency. Or Huey Long's governorship. It appears obvious to me that the word "ever" in your statement that "no progressive change has ever resulted from the election of a so-called leftist politician" should be struck out. There have evidently been numerous "leftist" politicians elected who have made progressive policy decisions without being forced into it by grassroots movements.

   \- Chris

2. [Vincent Geddes](http://blogs.gnome.org/vgeddes) says:[13 April 2008 11:36 AM](#32acf54062fc9fcb7c64420109744f6848b1a637)

   Well, the short story is that most Smalltalk VMs only support green threads (often of co-operative nature).

   The Smalltalk philosophy is that the VM should be as simple as possible, and ideally written in Smalltalk itself (meta-circularity). Making a Smalltalk VM fully multi-threaded would add a hideous amount of complexity, and make the system brittle and hard to understand. Much of complexity arises out of designing a parallel garbage collector, and making it work well with the rest of the system.

   To support this state of affairs, most Smalltalkers (as I perceive) have decided that native threads are "evil", and that it is often better to use other forms of concurrency.

   Now for a shameless plug! I am writing a Smalltalk VM (http://launchpad.net/goo-smalltalk) which I hope to turn to into a development environment for GNOME. Still in early development though, not much to show yet. However, I do know that I won't support native threads!

   Vincent

3. [Andy Wingo](http://wingolog.org/) says:[13 April 2008 8:59 PM](#d5a92ae65156d80add2be6dc725b030b296ac329)

   Hi Chris,

   Thanks for commenting. While it could be that I still have an insufficiently nuanced perspective, for now I stand by the statement. Forgive me for being ignorant about Huey Long, but, regarding Roosevelt, Howard Zinn says it better than I. In a recent article entitled [Election Madness](http://www.progressive.org/mag_zinn0308):

   > Let's remember that even when there is a "better" candidate (yes, better Roosevelt than Hoover, better anyone than George Bush), that difference will not mean anything unless the power of the people asserts itself in ways that the occupant of the White House will find it dangerous to ignore.
   >
   > The unprecedented policies of the New Deal—Social Security, unemployment insurance, job creation, minimum wage, subsidized housing—were not simply the result of FDR's progressivism. The Roosevelt Administration, coming into office, faced a nation in turmoil. The last year of the Hoover Administration had experienced the rebellion of the Bonus Army—thousands of veterans of the First World War descending on Washington to demand help from Congress as their families were going hungry. There were disturbances of the unemployed in Detroit, Chicago, Boston, New York, Seattle.
   >
   > In 1934, early in the Roosevelt Presidency, strikes broke out all over the country, including a general strike in Minneapolis, a general strike in San Francisco, hundreds of thousands on strike in the textile mills of the South. Unemployed councils formed all over the country. Desperate people were taking action on their own, defying the police to put back the furniture of evicted tenants, and creating self-help organizations with hundreds of thousands of members.
   >
   > Without a national crisis—economic destitution and rebellion—it is not likely the Roosevelt Administration would have instituted the bold reforms that it did.

   The rest of the article is worth it as well.

4. GimmeABreak says:[14 April 2008 7:23 AM](#6df68303f4c1964d6122180f87598bffdb330555)

   "But let us not forget that even the conservative UNICEF estimates that 500,000 Iraqis, mostly children, died as a result of the Clinton-imposed Iraq sanctions."

   No. They died because the ruthless dictator who controlled the country decided that piling up weapons, building up his personal riches and oppressing the shia Iraquis were all very important activities, much more so than buying medicines and caring for his own people. Remember he also wasted for military expenses the income from oil sales he was supposed to use for civilian only purposes.

   "The real number is assuredly higher"

   Huh, make that 50 million if you want, I wouldn't feel so "assured" especially if you consider that Saddam used this very issue as propaganda (there were reports of families deprived of they're deceased little kids corpses to be shown as victims of the infamous embargo). It's quite sad that people still support the cynic propaganda of a dead dictator.

   "This is Genocide, by "both" parties."

   I'm afraid you don't really know what "genocide" means.

   GimmeABreak

5. James says:[14 April 2008 3:09 PM](#67d2325bb4fb5b6a4a8559730aef9b9c9aa241d1)

   For unambiguity, I'm going to refer to Mako as mako.cc from now on. Partially because you note Mako has no google juice, partially because it amuses me to equate URIs to people.

6. [Dafydd Harries](http://dgh.livejournal.com) says:[13 May 2008 9:03 PM](#c807934d53fd32df11f32e1088b3c090cbc0da6e)

   Washing Machine is amazing. Diamond Sea alone makes the album. I also particularly like Unwind and Skip Tracer.

Comments are closed.
