---
title: 'Made in China: Dedication'
url: https://www.bunniestudios.com/blog/2007/made-in-china-dedication/
published: "2007-07-13T10:22:41Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=187
---

# Made in China: Dedication

This story needs a little background to fully appreciate.

There is a microphone in the new chumby. The particular microphone I decided to use has an integral pre-amp FET (an [“electret”](http://en.wikipedia.org/wiki/Electret_microphone) type). As such, it must be inserted in the correct orientation with respect to the circuit so the FET receives a proper bias current.

The first samples I got back from the factory had the microphone in backwards. So I called the factory and told them that they need to reverse the polarity of the microphone. I was going to come in for a visit the next week, and I wanted to see corrected samples. When I got to the factory and tested the microphone, I found out to my dismay that the microphones were *still* not working.

How could this be? There are only two ways to put a microphone in.

It turns out that they had two operators on the line assembling the microphone. One solders the red and black wires to the microphone. The next solders these red and black wires onto the circuit board. The operators were told to reverse the order, and both of them dutifully complied…giving me a microphone that was still soldered in backwards, but with the color of the wires swapped. This is actually a pretty typical story for problems in China…

At any rate, this leads up to the real point of this post. The next day, we had a first pilot run of 450 circuit boards scheduled up. Everything had to go perfectly for us to be on schedule. We had stencils rebuilt (we were debugging a yield issue with the QFN packaged audio CODEC as well) and ready by around noon, and around 6PM I had the first boards in my hands to test. As I was running the final factory test, the device failed again–at the microphone.

Needless to say, this was not a happy moment for anybody in the factory, as the factory is liable for any manufacturing defects. I donned my smocks and marched onto the line to start debugging the problem.

![](http://bunniestudios.com/blog/images/mic_debugging.jpg)

That’s me at 3 AM later that day. I’m still in the factory, and so is every manager and tech involved in the project. The pressure was fairly enormous–right next to us was a line churning out 450 potentially defective circuit boards, and I was unwilling to pull the plug on it because I didn’t know what the root cause was yet, and we had to stay on schedule.

I literally had a panel of factory workers standing by me the entire night to help me with anything I needed–soldering irons, test equipment, more boards, X-ray machines, microscopes. The remarkable thing is that not a single one hesitated for a moment, not a single one complained, not a single one lost focus on the problem, people cancelled dinner plans with friends without even batting an eyelash. If they weren’t needed that moment they were busy overseeing other aspects of the project. And this went on until 3 AM. With one exception, I hadn’t seen blind dedication like this since I worked with the autonomous underwater robotics team at MIT.

Embarassingly, the problem wasn’t their fault in the end. It was the new firmware release that was given to me earlier that day by the team in the US–it had a bug that disabled the microphone due to a hack that was accidentally checked into the build tree.

I think even more impressive is that when they found this out, nobody was angry, nobody complained (well, the sales lady gave me a hard time but I felt I deserved it; nevertheless, she was kind enough to accompany me on the production line all night long and be my translator, since my Mandarin isn’t up to snuff). They were simply relieved that it was not their fault. We all parted ways and I came back into the factory the next day at 11 AM after a good nights sleep. I saw Christy and I asked her when she came in. She told me she always has to report in by 8 AM. Now, I was starting to feel really bad–she stayed up late because of our bug and she came in early while I slept in. I asked her why she stayed up so late even though she knew she had to report to work at 8 AM–she could have gone home and we could have continued the next day. She just smiled and said “it’s my job to make sure this gets done, and I want to do a good job.”

![](http://bunniestudios.com/blog/images/mic_pcb_team.jpg)

On the right is Christy (PM), and to her left is Xiao Li (QA manager). The equipment they are observing is the chumby production tester, a bed-of-nails device developed in the US by me that facilitates the automatic firmware programming, unique keying, and testing of every circuit board.

Here’s another interesting story. On our way out of the factory floor one day, Xiao Li asked me what does a chumby do? Well, I don’t speak chinese very well, and she doesn’t speak english very well either, so I decided to start with a few basic questions.

I asked her if she knew what the world wide web was. She said no.

I asked her if she knew what the internet was. She said no.

I was stunned. Here is a girl who is an expert in building and testing computers–I mean, on some projects she has probably built PCs and booted Windows XP a hundred thousand times over and over again (god knows I heard that darn startup sound a zillion times that night on the factory floor, as right next to me was a bank of final test stations for ASUS motherboards)–yet she didn’t know what the internet was. I had taken it for granted that if you touched a computer today, you were also blessed by the bounties of the internet. I felt like a bit of a spoiled snob and a pig all at once for forgetting that she probably couldn’t afford a computer, much less broadband internet access. If she were given the opportunity, she was certainly smart enough to learn it all, but she’s busy making money that she’s probably sending back to her family at home.

How do you describe the color blue to the blind? In the end, the best I could do was to tell her it was a device for playing games.
