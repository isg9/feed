---
title: PLDI in Beijing
url: https://blog.regehr.org/archives/727
published: "2012-06-17T05:57:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=727
---

# PLDI in Beijing

\[nggallery id=53\]

PLDI 2012 was in Beijing earlier this week. Unfortunately I had only one full day to be a tourist; it would have been nice to bail out of the conference for another half day to see more stuff but that didn’t end up happening. My student Yang Chen went to college in Beijing and knew his way around; he showed me and my colleague Mary Hall around on our first day in Beijing. We walked around the olympic village (right next to the conference hotel) and then took the subway to Tiananmen Square. I liked the way the maps on the subway trains show your current location. The system is huge, probably comparable with London’s; it’s hard to believe it had just two lines in the 1990s.

We spent a half-day in the Forbidden City; we got lucky and had weather in the mid 80s with clear skies. Then we walked around a while and ate Szechuan beef, frogs, and eels. I was a little surprised how spicy everything was, but really it was perfect and didn’t get in the way of the flavors at all. I was a bit worried that my poor chopstick technique would make me look silly, but the Chinese attitude seems to be very practical. I’d have liked to spend a lot more time just walking around but at some point in the afternoon jet lag caught up with us and we took a taxi back to the hotel. My almost-former student Lu Zhao took us out for dinner and the food was superb.

One of the main reasons I like going to conferences is that while I’m naturally monomaniacal, talking to people and listening to talks can kind of shake my brain out of whatever rut it has gotten itself into. This isn’t something that has to happen all that often, but it feels important. Something funny about going to conferences these days is that often there are so many people I want to talk to that I end up missing presentations that I’d have liked to attend. This is in sharp contrast to my experiences at conferences 10 or 15 years ago where I basically didn’t know anybody.

Operating behind the Chinese firewall was annoying, mainly because Google seemed to be probabilistically blocked. Also, it took a while to deprogram myself from clicking on Twitter and Facebook any time I got a bit bored. That was a habit that needed breaking anyway. One of the interesting quirks was fine-grained blocking of Wikipedia pages. For example, the main entry for China loaded perfectly well but I couldn’t get to any page regarding the history of China.

One of the most pleasant things about this trip was learning about a number of Csmith users who I hadn’t known about. The tool appears to be pretty well-known these days and I guess it’s easy enough to use that people actually use it. Open source is weird that way: there are these silent users that you never hear about except by random chance. One user, Wei Huo from the Chinese Academy of Science, invited me to visit and give a talk about Csmith; this was great to do. There are also some Csmith users at Tsinghua and two of my students–Xuejun Yang, the main Csmith developer, and Yang Chen–gave talks there on the day that I flew home. A guy from a compiler consulting company told me that they’re getting a lot of bug reports from Csmith, which means that at least some embedded compiler practitioners are aware of Csmith. This is important first because these practitioners are highly non-academic, and second because it is the embedded compiler domain where I think tools like Csmith are most important. In other words, it’s not like GCC for x86 is so buggy that it’s hard to use–but some fraction of embedded compilers are buggy enough to make everyday development painful.

I gave the talk on [C-Reduce](http://embed.cs.utah.edu/creduce/), which was fun since us mid-career professors don’t get to do this very often–it is almost always the students who give the talks. My reasoning was that even though this paper had some student authors, C-Reduce has been my pet project for several years and I did basically all of the heavy lifting to get our submission out the door. However, between the submission deadline last November and now, Yang Chen did a ton of work on source to source transformations for C-Reduce and now his code dominates mine by maybe 10x. So probably I should have let him give the talk, but this possibility didn’t even occur to me until it was too late.
