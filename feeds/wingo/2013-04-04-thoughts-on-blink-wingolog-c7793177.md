---
title: thoughts on blink — wingolog
url: https://wingolog.org/archives/2013/04/04/thoughts-on-blink
published: "2013-04-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/04/04/thoughts-on-blink
---

# thoughts on blink — wingolog

## [thoughts on blink](/archives/2013/04/04/thoughts-on-blink)

4 April 2013 6:43 AM

- [webkit](/tags/webkit)
- [igalia](/tags/igalia)
- [blink](/tags/blink)
- [google](/tags/google)
- [chromium](/tags/chromium)

So [Chromium forked WebKit](http://www.chromium.org/blink) again! You've probably seen some articles on it already, but [Alex Russell's piece](http://infrequently.org/2013/04/probably-wrong/) is pretty good.

In retrospect the split was easy to see coming, but I can't help but feel bittersweet about it. The announcement and Alex's article both cite technical reasons for the fork, but to me it seems to be ultimately a human failure. So the Chromium team saw problems in WebKit and agreed on a solution that will help them achieve their goals faster: that's great! But why were they unable to do so within WebKit itself?

There are technical differences, yes, but those could be resolved by human action (as they now will be, by humans, under the Blink aegis). No, to me the fundamental problem was a lack of the [conditions for consensus](http://www.seedsforchange.org.uk/consensus#conditions) within the WebKit project, and for that there is ample blame to go around.

In the best of events this split will end up with both Blink and WebKit as more technically consistent, nimble projects. I would also hope that this split serves as a wakeup call to some of the decision-making processes within WebKit. However it is easy to be pessimistic in this regard -- the Apple-Google dynamic has been great for WebKit as an open project, opening up discussions and collaboration possibilities, especially among folks that don't work for the two major corporations. Both projects will have to actively make an effort to remain open, because passivity will favor the structural tendency to work in-house.

I should probably mention the obvious thing, that [Igalia](http://igalia.com) will work with both projects, of course. We'll get our WebKit patches into Blink, and vice versa. The message for customers doesn't change much, either -- most customers have already chosen a particular WebKit port, and those that used Chromium will now use Chromium on Blink.

For those projects that haven't chosen a WebKit port yet, the fundamental decision has always been WebKit2/JSC versus Chromium/V8, and that hasn't changed too much from this announcement. It's true that WebKit2 is a less mature multi-process implementation than Chromium, but it is getting a lot of investment these days, so it's fair to assume that technically that the two approaches will work just as well within a few months. The same goes for JSC versus V8.

Finally, a meta-note. Sometimes these press releases catch us up in their whirlwind and we're left with a feeling of anxiety: "Google just released AngularJS, now I need to rewrite all my apps!?" Well, maybe ;) But I find a bit of dispassion is appropriate after a piece of news like this. Good luck to both projects, and see you on the mailing lists.

## related articles

- [arrow functions coming to chrome 45!](/archives/2015/06/18/arrow-functions-coming-to-chrome-45)
- [inside javascriptcore's low-level interpreter](/archives/2012/06/27/inside-javascriptcores-low-level-interpreter)
- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [webkittens! lexical scoping is in danger!](/archives/2011/12/02/webkittens-lexical-scoping-is-in-danger)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [nominal types in webassembly](/archives/2026/03/10/nominal-types-in-webassembly)

### One response

1. Gustavo Noronha says:[11 April 2013 11:19 PM](#0be27941f1701702642d4b6dfedd6648ac12c041)

   Your thoughts echo my own quite a bit. I also saw it coming, I also think it's been human failure at the heart of the matter, but I also think the technical reasons are sound. The human failure started with Chromium people building their own JS engine and their own multiprocess architecture instead of pushing WebKit in that direction.

   From the Hacker News discussion, it looks like it deepened by the Chromium people not agreeing to push their multiprocess architecture upstream (or down the stack) and then with Apple deciding to go their own way - which as usual ended up being more welcoming to other ports. What ensued was pretty much expected from my point of view, the design/architecture disagreements were planted by these early seeds of not trying harder to work together.

   I'm an optimist, though, and think this split has at least as much potential of doing good to the web in general, WebKit and Chrome as it has of doing good, though I think it's likelyl both Blink and WebKit2 will go faster in terms of development.

Comments are closed.
