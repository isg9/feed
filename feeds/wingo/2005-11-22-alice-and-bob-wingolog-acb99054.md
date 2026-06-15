---
title: alice and bob — wingolog
url: https://wingolog.org/archives/2005/11/22/alice-and-bob
published: "2005-11-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/11/22/alice-and-bob
---

# alice and bob — wingolog

## [alice and bob](/archives/2005/11/22/alice-and-bob)

22 November 2005 8:31 PM

- [gstreamer](/tags/gstreamer)

**synchronicity**

Imagine Alice and Bob are in two buildings overlooking the same street. Each of them has a camera and occasionally takes polaroid pictures of the street. When Alice takes a picture, she writes the current time, according to her watch, on the back of the photo. Bob does the same with his photos and his watch. Later on, Alice wants to get the pictures made by both of them and put them in real time order, so she can see what is happening in the street from the two perspectives. Artsy types, wanting the zaniest things.

However, Alice knows that their watches are wrong -- they are hours apart, and one of them is running faster than the other. The other problem is that she doesn't know which one is faster, or by how much, and Bob is in another building so they can't see or hear each other. How then to put the pictures in order?

Given this problem, the solution I settled on was to treat one of the watches as the "correct" time, and to have the other one try to match its time to the "correct" time. For example, if Bob is the one with the correct time, Alice could send messengers to him to ask him the time. Whenever a messenger reaches Bob, he could write the time on a piece of paper and send it back.

Sending a message takes time though, so Alice doesn't know what her time is when Bob records his time. She then settles on a different plan: she writes down her time on a message before sending it to Bob. Bob writes down his time when he gets the message, and then Alice writes down the time when she gets the message back. Then she guesses that Bob wrote down his time in the middle of her times, if the message took the same time to get there as it did to come back.

Alice sends a few messages, and can check whether the difference between her time and Bob's time is increasing or decreasing, to see which clock is going fast. Also that way she can average out the different messengers' trips, because some of them will be faster, some slower, and maybe some messenger finds something more interesting to do than run around with pieces of paper.

In the end she can adjust her time with a rate difference, to account for the different speeds, and an absolute difference, to account for the different starting times.

Alice's problem is the same as capturing video from different machines and trying to mix them so they are in sync. One machine should have the master clock, and the others should try to analyze the difference between their own clock and the master via sending packets over the network. Conversely, the same issues come up if you are sending video to 9 computers with monitors, trying to form a gigantic screen -- all parts of the video should be shown at exactly the same time, which means the clocks must be synchronized, even if they run at slightly different rates.

In the end you end up something like [this](//wingolog.org/pub/network-clock.png), where "Local time" is Alice's time, "Remote time" is Bob's time, and "Network time" is what Alice thinks Bob's time is.

I worked out this [synchronization algorithm](//wingolog.org/archives/2005/07/08/updates) a while ago, but just got around to implementing it in GStreamer last week. It rocks. You make one pipeline's clock export a NetTimeProvider interface, which does Bob's job: receiving UDP packets with one time value, appending Bob's time to them and sending them back.

Then on the other pipeline you instantiate a network client clock and set it on the pipeline. When your webcam captures frames, it timestamps them using the network clock. Very neat. I'll integrate this into Flumotion later this week.

## related articles

- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [catching memory leaks with valgrind's massif](/archives/2008/05/05/catching-memory-leaks-with-valgrinds-massif)
- [why god made buttermilk](/archives/2006/12/17/why-god-made-buttermilk)
- [radical adults](/archives/2006/12/15/radical-adults)
- [cypherpunks writecode](/archives/2006/07/21/cypherpunks-writecode)
- [plane from spain](/archives/2005/12/21/plane-from-spain)

### 2 responses

1. [turkey preparations -- andy wingo](http://wingolog.org/?p=138) says:[24 November 2005 4:34 AM](#30)

   \[...\] alice and bob notes \[...\]

2. [pos-pavo -- andy wingo](http://wingolog.org/?p=139) says:[9 December 2005 1:57 AM](#31)

   \[...\] Did somebody say work? Because I was just thinking that! Lately I’ve been hacking the clock synchronization stuff into flumotion. It is looking most excellent. It’s tough to do a job properly, such that no one will have to come and clean up after you later, but it’s also satisfying. I am pleased with work right now. \[...\]

Comments are closed.
