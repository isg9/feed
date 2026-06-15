---
title: ffconf 2014 — wingolog
url: https://wingolog.org/archives/2014/11/09/ffconf-2014
published: "2014-11-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/11/09/ffconf-2014
---

# ffconf 2014 — wingolog

## [ffconf 2014](/archives/2014/11/09/ffconf-2014)

9 November 2014 5:36 PM

- [javascript](/tags/javascript)
- [javascriptcore](/tags/javascriptcore)
- [spidermonkey](/tags/spidermonkey)
- [v8](/tags/v8)
- [self-hosting](/tags/self-hosting)
- [bootstrapping](/tags/bootstrapping)
- [igalia](/tags/igalia)

Last week I had the great privilege of speaking at [`ffconf`](http://2014.ffconf.org/) in Brighton, UK. It was lovely. The town put on a full demonstration of its range of November weather patterns, from blue skies to driving rain to hail (!) to sea-spray to drizzle and back again. Good times.

The conference itself was quite pleasant as well, and from the speaker perspective it was amazing. A million thanks to Remy and Julie for making it such a pleasure. `ffconf` is mostly a front-end development conference, so it's not directly related with the practice of my work on JS implementations -- perhaps you're unaware, but there aren't so many browser implementors that actually develop content for their browsers, and indeed fewer JS implementors that actually write JS. Me, I sling C++ all day long and the most JavaScript I write is for tests. When in the weeds, sometimes we forget we're building an amazing runtime and that people do inspiring things with it, so it's nice to check in with front-end folks at their conferences to see what people are excited about.

My talk was about the part of JavaScript implementations that are written in JavaScript itself. This is an area that isn't so well known, and it has its amusing quirks. I think it can be interesting to a curious JS hacker who wants to spelunk down a bit to see what's going on in their browsers. Intrepid explorers might even find opportunities to contribute patches. Anyway, nerdy stuff, but that's basically how I roll.

The slides are here: [without images (350kB PDF)](//wingolog.org/pub/ffconf-2014-slides-plain.pdf) or [with images (3MB PDF)](//wingolog.org/pub/ffconf-2014-slides.pdf).

I haven't been to the UK in years, and being in a foreign country where everyone speaks my native language was quite refreshing. At the same time there was an awkward incident in which I was reminded that though close, American and English just aren't the same. I made this silly joke that if you get a [polyfill](https://en.wikipedia.org/wiki/Polyfill) into a JS implementation, then shucks, you have a "gollyfill", 'cause golly it made it in! In the US I think "golly" is just one of those milquetoast profanities, "golly" instead of "god" like saying "shucks" instead of "shit". Well in the UK that's a thing too I think, but there is also another less fortunate connotation, in which "golly" as a noun can be a racial slur. [Check the Wikipedia](https://en.wikipedia.org/wiki/Golliwog) if you're as ignorant as I was. I think everyone present understood that wasn't my intention, but if that is not the case I apologize. With slides though it's less clear, so I've gone ahead and removed the joke from the slides. It's probably not a ball to take and run with.

However I do have a ball that you can run with though! And actually, this was another terrible joke that wasn't bad enough to inflict upon my audience, but that now chance fate gives me the opportunity to use. So never fear, there are still bad puns in the slides. But, you'll have to click through to the PDF for optimal groaning.

Happy hacking, JavaScripters, and until next time.

## related articles

- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [state of js implementations, 2014 edition](/archives/2014/12/09/state-of-js-implementations-2014-edition)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [optimizing let in spidermonkey](/archives/2013/12/18/optimizing-let-in-spidermonkey)

### 3 responses

1. [Geoffrey Sneddon](http://gsnedders.com) says:[10 November 2014 0:13 AM](#00f0b0be63183815416c3fbe2b2da61fe863d902)

   FWIW, as someone who is British and lived here almost all my life, I've never heard of "golly" being an insult. "Golliwog" is questionable as a name of a toy, and a "wog" is outright offensive. The Wikipedia article only suggests "golliwog" and "wog" as offensive as far as I can tell, too?

2. [wingo](http://wingolog.org/) says:[10 November 2014 10:08 AM](#cfdeff385cccb3fe4443b9091ebeed4eafc716eb)

   Hi Geoffrey, I don't want to dwell on the topic -- me as a white dude tossing around racial slurs as if they're only of academic interest is a bit shitty really. However the concern was mentioned to me, verified by others, and also discussed a bit in the Wikipedia article. Let's move on, shall we -- my intention was only to indulge my punning habit :)

3. Marc Driftmeyer says:[12 November 2014 0:36 AM](#359b51b5876ef1a9740ce396d4c5ffbce214178e)

   Golly in America \[as an American\] is a kindly 50/60s/70s slang for amazement, most famously used by Gomer Pile on The Andy Griffith Show.

   http://articles.orlandosentinel.com/2013-01-30/entertainment/os-jim-nabors-marries-male-partner-20130130\_1\_jim-nabors-gomer-pyle-gay-marriage

   It's most commonly thought of, \`\`imagine that!'' or , \`\`amazing! I never knew that.''

Comments are closed.
