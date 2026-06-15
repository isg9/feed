---
title: a schemer at jsconf.eu — wingolog
url: https://wingolog.org/archives/2011/10/04/a-schemer-at-jsconf-eu
published: "2011-10-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/10/04/a-schemer-at-jsconf-eu
---

# a schemer at jsconf.eu — wingolog

## [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)

4 October 2011 6:23 PM

- [javascript](/tags/javascript)
- [scheme](/tags/scheme)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [v8](/tags/v8)
- [gnome](/tags/gnome)
- [gnu](/tags/gnu)

Yow! I am just back from [JSConf.eu](http://jsconf.eu/), a lovely & lively gathering of JavaScript hackers in sunny Berlin. And it was sunny, this foreign land, and foreign indeed: as my long-time readers will know, I am not a JavaScript hacker, but a Schemer with a C problem. JSConf.eu was a pleasant place to visit.

It turns out that I wasn't the only one there in the tourist condition; all of the other implementors seemed to be in a similar situation. The Mozilla folks look at themselves as "platform engineers". V8 hacker Vyacheslav Egorov has a thing for Oberon; I'm sure he'll tell you about it if you ask. And so on. Indeed I think that of the folks that were there, Brendan is the only one implementing JavaScript in JavaScript. [*Do You Speak It!*](http://nelm.io/shop/)

**great job, organizers!**

But besides giving me the opportunity to gossip about garbage collection and type feedback, JSConf.eu was a well-organized, energetic event, and I say that as one who has been to dozens of conferences in the last few years.

There was great sound and video, of course. It looked like they did a pretty good job with the recording, so the videos should be online at some point. In the downstairs room, they had [Anna Lena Schiller](http://www.annalenaschiller.com/) drawing live "graphic recordings" as the speakers spoke, on large sheets of glossy paper, in color, illustrating the main ideas of the talks. As the weekend progressed, all of these pieces were mounted on the wall for people to see. In the end they will be scanned and uploaded, and the originals mailed to the speakers. It was amazing, and an impressive amount of work.

It's silly, but let me also mention the food: there were fresh-made frozen-yogurt smoothies with brownie toppings. There were fresh-made croissants and bread, and a plate of some dozen or so French cheeses, and made-to-order breakfast, and plates of tarts, and the bar was open all day long, with apfelschorle and club-mate and beer and coffee, and there was an espresso stand, and everything was free. There was lunch and dinner at the venue and free parties before, during, and after. The only caveat was that the vegetarian options were relatively few. Still, though way to raise the bar, JSConf.eu!

**hypersensitivity?**

I could be overly sensitive about this, but I would like to put one thing out there. There were a number of sexuality-related aspects of this event that I did not feel comfortable with. There were a couple speakers that made live-chat applications with node.js, and demoed them to the room, and a few anonymous penis jokes ended up on them. There was a speaker that made a sexual joke about `Allow: GET, HEAD` in HTTP headers. Most of all though, they had the (very talented) caberet singer [Mandy Lauderdale](http://www.mandylauderdale.com/) there, who started with a "Brendan Eich Fanclub" thing, then put on a show the next night at the party.

Please understand me when I say that I have no problem with sexuality in general -- it is fun, subversive, and in any case a part of life. However in the programming world we have a gender-balance problem, and I don't think that incorporating sexuality into our culture helps that. In the particular case of Mandy's (talented and sometimes funny) performance, I get the feeling that we're promoting a "boy's club" atmosphere where it becomes OK to make penis jokes.

Like I said, I'm just putting that out there. Feel free to disagree, politely of course.

**talks**

The first talk I saw was by [Dean McNamee](http://www.deanmcnamee.com/) of [plask](http://www.plask.org/), a non-browser JS environment for making art. Plask helps make graphical art by providing 2D and 3D drawing primitives to JS. It also integrates with other devices via MIDI and OSC, and can be sequenced over MIDI (or OSC, I presume).

Dean's message was that we need to stop caring so much about the "how" and care more about the "what" and "when". There was no code in the presentation, only demos of what Plask has done; and indeed Dean was quite honest that it was the crappiest code he has ever written. But the demos were pretty awesome.

There are obviously some problems with that approach to software, that it is difficult to share or to form a piece of someone else's work. However it is equally true that many of us spend a lot of time working on things that people will never see, and that sometimes we should just take shortcuts and move on.

Another interesting talk was by [Peter van der Zee](http://qfox.nl/) on JavaScript tools. Peter wrote a JS parser in JS, and an in-browser editor to overlay that tool information and utility onto a piece of JS code. It can provide many warnings, do type inference, and similar things. It can do some automated refactorings, like var hoisting, extract-method, and minification. I wonder if some more principled analysis algorithms like [*k* CFA](http://matt.might.net/articles/implementation-of-kcfa-and-0cfa/) would provide any features above what he is able to do with his fixed-point constraint solver. Still, neat stuff; check it out [on github](http://github.com/qfox/zeon).

David Mandelin and David Anderson of Mozilla gave a nice [introduction to what JIT compilers can do these days](http://www.slideshare.net/iamdvander/js-math-jiting-jsconfeu) in JavaScript, at least at a very local level. The presentation reminded me of Mandelin's old [pjit](https://bugzilla.mozilla.org/show_bug.cgi?id=646923) project, to figure out what sorts of optimizations would be necessary to yield the smallest assembly code. My only criticism is that some of their assumptions were specific to the [nan-boxing](//wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations) approach. They did include a humble nod at the end that there was still a ways for IonMonkey to go in order to reach what Crankshaft does, which was appropriate IMO.

Brendan Eich gave a talk at the end of the day about the coming ES6. Frankly I have no idea how people could understand the things he spoke of without following `es-discuss`, and most people I asked afterwards were really skeptical about such things as the "monocle-mustache" operator (that's `.{` for those of you following along at home). That said, I do like the triangle operator ( `|>`) and the potential block-lambda syntax. Slides are not up yet, unfortunately.

`. . .`

The next day, I was pleased to find myself sitting with GNOME hackers [Henri Bergius](http://bergie.iki.fi/) and [Garrett LeSage](http://garrettlesage.com/). We all agreed that it was great to be in such a dynamic, energetic conference, and that JSConf.eu was much more lively than recent free software conferences we had been to. However it must be said that JS programmers are at an early stage of free software development in many ways, in that there are many more single-person projects than we have in GNOME (for example). It could be said that our use of C in GNOME forces you to collaborate because you can't build very much (hah!), but large, stable projects with many contributors are relatively rare.

Also, I thought there is a general low esteem among the JS folk for copyleft and privacy, and for software freedom in general. There are exceptions of course, but JS is closely tied to environments with lots of venture capital, so it is to be expected that we get divergence of interests between users and developers. I think this is an area in which GNU folk have something important to say: that it's not just about the tech, it's about the freedom too.

On that note, my Igalia colleague [Eduardo Lima](http://blogs.free-social.net/elima/)'s talk on JavaScript in GNOME was well-received. Eduardo talked about the pieces that allow a good JavaScript hacking experience to be had in GNOME, and even did some live-coding. Hopefully he'll blog about his talk soon :-)

I really enjoyed [Tom Robinson](http://tlrobinson.net/)'s talk about compilers and JavaScript. It was about compilers that produce or consume JavaScript. I suppose that in some ways compiling DSLs to JavaScript in an in-browser compiler that passes the result to `eval` is somewhat like macros, though it is hard to compose with other parts of the target language like lexical scope. Interesting stuff though, and I hope he puts up his slides soon.

Finally, and we're getting to the end here -- I did not intend to write so much! -- V8 hacker [Erik Corry](http://twitter.com/erikcorry) had a great talk about garbage collection. V8 always had a generational, moving collector, which is great for most purposes, but struggles with large heaps, because collecting the old generation required a stop-the-world mark and sweep. Their new GC can incrementally collect the old generation, leading to reductions in pause times and higher sustained allocation rates for some problems (like the [splay](http://v8.googlecode.com/svn/data/benchmarks/v6/splay.js) benchmark). There's lots of tricky write-barrier stuff going on there, and they're just now working out the details, but it will arrive to a Chrome near you shortly.

As part of Erik's work, he had enhanced the V8 engine to output heap profiling information on a TCP port. He could then run a node.js process to consume this information and output a graphical representation of the heap in a browser. It shows the new space filling up, scavenging, and promoting data into various parts of the old space. It also shows the old space being incrementally marked and swept, and has a visualization of time spent in the collector versus the mutator. This last visualization nicely showed that the new GC allows the splay benchmark to make forward progress even with a large rate of tenuring into the old generation.

I'm sure the Google folks will make their press release about this as soon as things settle down, but for now you can check it out in the `bleeding_edge` of V8. Have fun!

**fin**

A gist of [all the slides](https://gist.github.com/1256066) is up on github, and I suppose it will be updated as new slides are uploaded and videos come online.

There were a number of other talks I didn't get to mention, like Andreas Gal on PDF.js, or Alon Zakai on [Emscripten](https://github.com/kripken/emscripten/wiki). SQLite compiled from C to JS for the browser? What a world. Thanks again to the [JSConf.eu](http://jsconf.eu/) folks for such a great conference, and to [Igalia](http://www.igalia.com/) for sending me. Happy hacking!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [a closer look at crankshaft, v8's optimizing compiler](/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

### 5 responses

1. [Malte](http://twitter.com/cramforce) says:[4 October 2011 7:34 PM](#f55d678df6460fdc44063b53c544e18095556a44)

   Hey Andy,

   thanks for the feedback. Did not know you where there, would have loved to talk to you as I'm a big fan of your recent v8 blog articles. You've probably seen me at JSConf. I was the guy announcing most of the talks in track A.

   With respect to the hypersensitivity topic: We are actually very sensitive about diversity in tech and try to make everyone feel as comfortable as possible at our conference. With respect to Mandy: She wasn't a random pick, but rather the girlfriend of one of our previous speakers and this year's attendee. The performance is simply what she does (and as you found out she's really good at it). We talked to her for quite a while about the topic of diversity in tech and tried to walk the fine line the right way. I usually discuss things with my feminist mother. If she says I'm cool, I usually feel comfortable to go ahead with things.

   The world isn't perfect, so things will go wrong. Conferences only make this more visible. For next year, maybe we provide an API to resolve IP addresses of attendees to Twitter names, so that anonymous bullshit chats are no longer possible.

   Thanks again for the awesome feedback!

2. [Andy Wingo](http://wingolog.org/) says:[4 October 2011 8:53 PM](#80f28a92c7fbc1ca89523679b44f1d8c0e77dee5)

   Hi Malte! Thanks for stopping by. I should say again that I really enjoyed the conference, and enjoyed your stage presence as well. It was clear that you were having fun :-)

   I did float the concerns about Mandy by 2 or 3 other attendees and like you they also did not see an issue, so it's probably just me reacting too much.

   Enjoy the well-deserved post-conference rest :-)

   Andy

3. [Vyacheslav Egorov](http://mrale.ph) says:[4 October 2011 9:24 PM](#f9faeb08a7f5c10276a1ddbe068f51e2cc45ec18)

   It was nice meeting you, Andy!

   (\\* me and Wirth's languages are kinda in "it's complicated" and "I love to hate you" relationship \*)

4. [Karl G](http://gr.ayre.st/) says:[10 October 2011 4:09 PM](#d6a7097e2f4124c6cc9d05d951770f3991e74540)

   \> I wonder if some more principled analysis algorithms like kCFA would provide any features above what he is able to do with his fixed-point constraint solver.

   You may be interested in Dimitros Vardoulakis' CFA2. He did an internship at Mozilla last year and the result is DoctorJS, which is up on github. Alas, it's node.js based and hasn't been updated since last year so it's bitrotted due to node's api velocity. He has slides up and a video on microsoft research if you search around.

5. [Andy Wingo](http://wingolog.org/) says:[10 October 2011 4:37 PM](#df0c60b6d1780f0afd2a633f86f05ebdb4c7ff05)

   That's a fascinating link, Karl. Thanks!

Comments are closed.
