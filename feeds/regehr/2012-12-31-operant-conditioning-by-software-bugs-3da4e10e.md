---
title: Operant Conditioning by Software Bugs
url: https://blog.regehr.org/archives/861
published: "2012-12-31T07:38:47Z"
feed: regehr
guid: http://blog.regehr.org/?p=861
---

# Operant Conditioning by Software Bugs

![conditioning](https://blog.regehr.org/wp-content/uploads/2012/12/conditioning.jpeg) Have you ever used a new program or system and found it to be obnoxiously buggy, but then after a while you didn’t notice the bugs anymore? If so, then congratulations: you have been trained by the computer to avoid some of its problems. For example, I used to have a laptop that would lock up to the point where the battery needed to be removed when I scrolled down a web page for too long (I’m guessing the video driver’s logic for handling a full command queue was defective). Messing with the driver version did not solve the problem and I soon learned to take little breaks when scrolling down a long web page. To this day I occasionally feel a twinge of guilt or fear when rapidly scrolling a web page.

The extent to which we technical people have become conditioned by computers became apparent to me when one of my kids, probably three years old at the time, sat down at a Windows machine and within minutes rendered the GUI unresponsive. Even after watching which keys he pressed, I was unable to reproduce this behavior, at least partially because decades of training in how to use a computer have made it very hard for me to use one in such an inappropriate fashion. By now, this child (at 8 years old) has been brought into the fold: like millions of other people he can use a Windows machine for hours at a time without killing it.

Operant conditioning describes the way that humans (and of course other organisms) adapt their behavior in response to the consequences resulting from that behavior. I drink beer at least partially because this has made me feel good, and I avoiding drinking 16 beers at least partially because that has made me feel bad. Conditioning is a powerful guiding force on our actions and it can happen without our being aware of it. One of my favorite stories is where a psychology class trained the professor to lecture from the corner by paying attention when he stood in one part of the room and looking elsewhere when he did not. (This [may or may not be only an urban legend](http://msgboard.snopes.com/cgi-bin/ultimatebb.cgi?ubb=get_topic;f=99;t=000030), but seems plausible even so.) Surely our lives are filled with little examples of this kind of unconscious guidance from consequences.

How have software bugs trained us? The core lesson that most of us have learned is to stay in the well-tested regime and stay out of corner cases. Specifically, we will:

- periodically restart operating systems and applications to avoid software aging effects,
- avoid interrupting the computer when it is working (especially when it is installing or updating programs) since early-exit code is pretty much always wrong,
- do things more slowly when the computer appears overloaded—in contrast, computer novices often make overload worse by clicking on things more and more times,
- avoid too much multitasking,
- avoid esoteric configuration options,
- avoid relying on implicit operations, such as the fact that MS Word is supposed to ask us if we want to save a document on quit if unsaved changes exist.

I have a hunch that one of the reasons people mistrust Windows is that these tactics are more necessary there. For example, I never let my wife’s Windows 7 machine go for more than about two weeks without restarting it, whereas I reboot my Linux box only every few months. One time I had a job doing software development on Windows 3.1 and my computer generally had to be rebooted at least twice a day if it was to continue working. Of the half-dozen Windows laptops that I’ve owned, none of them could reliably suspend/resume ten times without being rebooted. I didn’t start this post intending to pick on Microsoft, but their systems have been involved with all of my most brutal conditioning sessions.

Boris Beizer, in his entertaining but long-forgotten book *The Frozen Keyboard*, tells this story:

> My wife’s had no computer training. She had a big writing chore to do and a word-processor was the tool of choice. The package was good, but like most, it had bugs. We used the same hardware and software (she for her notes and I for my books) over a period of several months. The program would occasionally crash for her, but not for me. I couldn’t understand it. My typing is faster than hers. I’m more abusive of the equipment than she. And I used the equipment for about ten hours for each of hers. By any measure, I should have had the problems far more often. Yet, something she did triggered bugs which I couldn’t trigger by trying. How do we explain this mystery? What do we learn from it?
>
> The answer came only after I spent hours watching her use of the system and comparing it to mine. She didn’t know which operations were difficult for the software and consequently her pattern of usage and keystrokes did not avoid potentially troublesome areas. I *did* understand and I unconsciously avoided the trouble spots. I wasn’t testing that software, so I had no stake in making it fail—I just wanted to get my work done with the least trouble. Programmers are notoriously poor at finding their own bugs—especially subtle bugs—partially because of this immunity. Finding bugs in your own work is a form of self-immolation. We can extend this concept to explain why it is that some thoroughly tested software gets into the field and only then displays a host of bugs never before seen: the programmers achieve immunity to the bugs by subconsciously avoiding the trouble spots while testing.

Beizer’s observations lead me to the first of three reasons why I wrote this piece, which is that I think it’s useful for people who are interested in software testing to know that you can generate interesting test cases by inverting the actions we have been conditioned to take. For example, we can run the OS or application for a very long time, we can interrupt the computer while it is installing or updating something, and we can attempt to overload the computer when its response time is suffering. It is perhaps instructive that Beizer is an expert on software testing, despite also being a successful anti-tester, as described in his anecdote.

![skinner](https://blog.regehr.org/wp-content/uploads/2012/12/skinner.png) The second reason I wrote this piece is that I think operant conditioning provides a partial explanation for the apparent paradox where many people believe that most software works pretty well most of the time, while others believe that [software is basically crap](http://www.hanselman.com/blog/EverythingsBrokenAndNobodysUpset.aspx). People in the latter camp, I believe, are somehow able to resist or discard their conditioning in order to use software in a more unbiased way. Or maybe they’re just slow learners. Either way, those people would make amazing members of a software testing team.

Finally, I think that operant conditioning by software bugs is perhaps worthy of some actual research, as opposed to my idle observations here. An HCI researcher could examine these effects by seeding a program with bugs and observing the resulting usage patterns. Another nice experiment would be to provide random negative reinforcement by injecting failures at different rates for different users and observing the resulting behaviors. Anyone who has been in a tech support role has seen the bizarre [cargo cult rituals](http://utcc.utoronto.ca/~cks/space/blog/unix/TheLegendOfSync) that result from unpredictable failures.

In summary, computers are Skinner Boxes and we’re the lab rats—sometimes we get a little food, other times we get a shock.

**Acknowledgment:** This piece benefited from discussions with my mother, who previously worked as a software tester.
