---
title: on gnu and on hackers — wingolog
url: https://wingolog.org/archives/2014/08/18/on-gnu-and-on-hackers
published: "2014-08-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/08/18/on-gnu-and-on-hackers
---

# on gnu and on hackers — wingolog

## [on gnu and on hackers](/archives/2014/08/18/on-gnu-and-on-hackers)

18 August 2014 9:07 PM

- [gnu](/tags/gnu)
- [fsf](/tags/fsf)
- [ghm](/tags/ghm)
- [munich](/tags/munich)
- [hacienda](/tags/hacienda)
- [knock](/tags/knock)
- [gnunet](/tags/gnunet)
- [guix](/tags/guix)
- [guile](/tags/guile)
- [misogyny](/tags/misogyny)
- [sexism](/tags/sexism)
- [intent](/tags/intent)

Greetings, gentle hackfolk. 'Tis a lovely waning light as I write this here in Munich, Munich the green, Munich full of dogs and bikes, Munich the summer-fresh.

Last weekend was the GNU hackers meeting up in Garching, a village a few metro stops north of town. Garching is full of quiet backs and fruit trees and small gardens bursting with blooms and beans, as if an eddy of Chistopher Alexander whirled out and settled into this unlikely place. My French suburb could learn a thing or ten. We walked back from the hack each day, ate stolen apples and corn, and schemed the nights away.

The program of GHM this year was great. It started off with a bang, as [GNUnet hackers Julian Kirsch and Christian Grothoff broke the story that the Five-Eyes countries (US, UK, Canada, Australia, NZ) regularly port-scan the entire internet, looking for vulnerabilities](http://www.heise.de/ct/artikel/NSA-GCHQ-The-HACIENDA-Program-for-Internet-Colonization-2292681.html). They then proceed to exploit those vulnerabilities, in regular hack-a-thons, trying to own as many boxes in as many countries as they can. They then use them as launchpads for attacks and for exfiltration of information from other networks.

The presentation that broke this news also proposed a workaround based on port-knocking, [Knock](https://gnunet.org/knock). Knock embeds the hash of a pre-shared key with some other information into the 32-bit initial sequence number of a TCP connection. Unlike previous incarnations of port-knocking, Knock also authenticates the first *n* payload bytes, so that the connection isn't vulnerable to hijacking (e.g. via GCHQ "quantum injection", where well-placed evil routers race the true destination server to provide the first response packet of a connection). Leaking the pwn-the-internet documents with Laura Poitras at the same time as the Knock unveiling was a pretty slick move!

I was also really impressed by Christian's presentation on the [GNU name system](https://gnunet.org/gns). GNS is a replacement for DNS whose naming structure mirrors our social naming structure. For example, `www.alice.gnu` would be my friend Alice, and `www.alice.bob.gnu` would be Alice's friend Bob. With some integration, it can work on normal desktops and mobile devices. There are lots more details, so check [gnunet.org/gns](https://gnunet.org/gns) for more information.

Of course, a new naming system does need some operating system support. In this regard Ludovic Courtès' update on [Guix](https://gnu.org/s/guix/) was particularly impressive. Guix is a Nix-like system whose goal is [reproducible](https://blog.torproject.org/blog/deterministic-builds-part-one-cyberwar-and-global-compromise), user-controlled GNU/Linux systems. A couple years ago I didn't think much of it, but now it's actually booting on raw hardware, not just under virtualization, and things seem to be rolling forth as if on rails. Guix manages to be technically innovative at the same time as being GNU-centered, so it can play a unique role in propagating GNU work like GNS.

**and yet.**

But now, as the dark clouds race above and the light truly goes, we arrive to the point I really wanted to discuss. GNU has a terrible problem with gender balance, and with diversity in general. Of about 70 attendees at this recent GHM, only two were women. We talk the talk about empowering users and working for freedom but, to a first approximation, it's really just a bunch of dudes that think the exact same things.

There are many reasons for this, of course. Some people like to focus on what's called the "pipeline problem" -- that there aren't as many women coming out of computer science programs as men. While true, the proportion of women CS graduates is much higher than the proportion of women at GHM events, so something must be happening in between. And indeed, the attrition rates of women in the tech industry are higher than that of men -- often because we men make it a needlessly unpleasant place for women to be. Sometimes it's even dangerous. [The incidence of sexual harassment and assault in tech, especially at events, is something terrible](http://geekfeminism.wikia.com/wiki/Timeline_of_incidents). Scroll down in that linked page to June, July, and August 2014, and ask yourself whether that's OK. (Hint: hell no.)

And so you would think that people who consider themselves to be working for some abstract liberatory principle, as GNU is, would be happy to take a stand against this kind of asshaberdashery. There you would be wrong. Voilà a timeline of an incident.

**timeline**

March 2014Someone at the FSF asks a GHM organizer to add an anti-harassment policy to GHM. The organizer does so and puts one on the web page, copying the text from [Libreplanet](http://libreplanet.org/)'s web site. The [policy posted](http://web.cvs.savannah.gnu.org/viewvc/ghm/policy.html?revision=1.5&root=ghm&view=markup) is:

> Offensive or overly explicit sexual language or imagery is inappropriate during the event, including presentations.
>
> Participants violating these rules may be sanctioned or expelled from the meeting at the discretion of the organizers.
>
> Harassment includes offensive comments related to gender, sexual orientation, disability, appearance, body size, race, religion, sexual images in public spaces, deliberate intimidation, stalking, harassing photography or recording, persistent disruption of talks or other events, repeated unsolicited physical contact, or sexual attention.

Monday, 11 August 2014The first mention of the policy is made on the mailing list, in a [mail](http://mid.gmane.org/201408111643.s7BGhPEn023124@jocasta.intra) with details about the event that also includes the line:

> An anti-harrasment policy applies at GHM: http://gnu.org/ghm/policy.html

Monday, 11 August 2014[A speaker writes the list](http://mid.gmane.org/871tsn53mi.fsf@gnu.org) to say:

> Since I do not desire to be denounced, prosecuted and finally sanctioned or expelled from the event (especially considering the physical pain and inconvenience of attending due to my very recent accident) I withdraw my intention to lecture "Introducing GNU Posh" at the GHM, as it is not compliant with the policy described in the page above.
>
> Please remove the talk from the official schedule. Thanks.
>
> PS: for those interested, I may perform the talk off-event in case we find a suitable place, we will see..

The [resulting thread](http://thread.gmane.org/gmane.org.gnu.ghm.general/614) goes totes clownshoes and rages on up until the event itself.Friday, 15 August 2014Sheepish looks between people that flamed each other over the internet. Hallway-track discussion starts up quickly though. Disagreeing people are not rational enough to have a conversation though (myself included).Saturday, 16 August 2014In the morning and lunch break, people start to really discuss the issues (spontaneously). It turns out that the original mail that sparked the thread was based, to an extent, on a misunderstanding: that "offensive or overly explicit sexual language or imagery" was parsed (by a few non-native English speakers) as "offensive language or ...", which people thought was too broad. Once this misunderstanding was removed, there were still people that thought that any policy at all was unneeded, and others that were concerned that someone could say something without intending offense, but then be kicked out of the event. Arguments back and forth. Some people wonder why others can be hurt by "just words". Some discussion of rape culture as continuum between physical violence and cultural tropes.

One of the presentations after lunch is by a GNU hacker. He starts his talk by stating his hope that he won't be seen as "offensive or part of rape culture or something". His microphone wasn't on, so once he gets it on he repeats the joke. I stomp out, slam the door, and tweet a [few](https://twitter.com/andywingo/status/500637145342935040) [angry](https://twitter.com/andywingo/status/500638045285416961) [things](https://twitter.com/andywingo/status/500638753330049024).

Later in the evening the presenter and I discuss the issue. He apologizes to me.Sunday, 17 August 2014A closed meeting for GNU maintainers to round up the state of GNU and GHM. No women present. After dealing with a number of banalities, we finally broach the topic of the harassment policy. More opposition to the policy Sunday than Saturday lunch. Eventually a proposal is made to replace "offensive" with "disparaging", and people start to consent to that. We run out of time and have to leave; resolution unclear.Monday, 18 August 2014GHM organizer updates the policy to remove the words "offensive or" from the beginning of the harassment policy.

I think anyone involved would agree on this timeline.

**thoughts**

The problems seen over the last week with this anti-harassment policy are **entirely to do with the men**. It was a man who decided that he should withdraw his presentation because he found it offensive that he could be perceived as offensive. It was men who willfully misread the policy, comparing it to such examples as "I should have the right to offend Microsoft supporters", "if I say the wrong word I will go to jail", and who raised the ignorant, vacuous spectre of "political correctness" to argue that they should be able to say what they want, in a GNU conference, no matter who they hurt, no matter what the effects. That they are able to argue this position from a status-quo perspective is the definition of privilege.

Now, there is ignorance, and there is malice. Both must be opposed, but the former may find a cure. Although I didn't begin my contribution to the discussion in the smoothest way, linking to a [an amusing article on the alchemy of intent](http://genderbitch.wordpress.com/2010/01/23/intent-its-fucking-magic/) that is probably misunderstood, it ended up that one of the main points was about intent. I know Ralph (say) and Ralph is a great person and so how could it be that anything Ralph would say could be a slur? You *know* he wouldn't mean it like that!

To that, we of course have to say that as GNU grows, not everyone knows that Ralph is a great person. In the end what would it mean for someone to be anti-racist but who says racist things all the time? You would have to call them racist, right? Or if you just said something one time, but refused to own up to your error, and instead persisted in repeating a really racist slur -- you would be racist right? But I know you... But the thing that you said...

But then to be honest I wonder sometimes. If someone repeats a joke trivializing rape culture, *after making sure that the microphone is picking up his words* \-\- I mean, that's a misogynist action, right? Put aside the question of whether the person is, in essence, misogynist or not. They are doing misogynist things. How do I know that this person isn't going to do it again, private apology or not?

And how do I know that this community isn't going to permit it again? That remark was made to a room of 40 dudes or so. Not one woman was present. Although there was some discussion afterwards, if people left because of the phrase, it was only two or three. How can we then say that GNU is not a misogynist community -- is not a community that tolerates misogyny?

Given all of this, what do you expect? Do you expect to grow GNU into a larger organization in the future, rich and strong and diverse? If that's not your goal, you are not my colleague. And if it is your goal, why do you put up with this kind of behavior?

The discussion on intent and offense seems to have had its effect in the removal of "offensive or" from the anti-harassment policy language. I think it's terrible, though -- if you don't trust someone who says they were offended by sexual language or imagery, why would you trust them when they report sexual harassment or assault? I can only imagine this leading to some sort of argument where the person who has had the courage to report such an incident finds himself or herself in a public witness box, justifying that the incident was offensive. "I'm sorry my words offended you, but that was not my intent, and anyway the words were not offensive." Lolnope.

There were so many other wrong things about this -- a suggestion that we the GNU cabal (lamentably, a bunch of dudes) should form a committee to make the wording less threatening to us; that we're just friends anyway; that illegal things are illegal anyway... it's as if the [Code of Conduct FAQ](http://www.ashedryden.com/blog/codes-of-conduct-101-faq) that Ashe Dryden assembled were a Bingo card and we all lost.

Finally I don't think I trusted the organizers enough with this policy. Both organizers expressed skepticism about the policy in such terms that if I personally hadn't won the privilege lottery (white male "western" hetero already-established GNU maintainer) I wouldn't feel comfortable bringing up a concern to them.

In the future I will not be attending any conferences without strong, consciously applied codes of conduct, and I enjoin you to do the same.

**conclusion**

Propagandhi, "Refusing to Be a Man", Less Talk, More Rock (1996)

There is no conclusion yet -- working for the better world we all know is possible is a process, as people and as a community.

To outsiders, to outsiders everywhere, please keep up the vocal criticisms. Thank you for telling your story.

To insiders, to insiders everywhere, this is your problem. The problem is you. Own it.

## related articles

- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [the user in the loop](/archives/2011/10/19/the-user-in-the-loop)
- [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)
- [gnu, gnome, and the fsf](/archives/2009/12/13/gnu-gnome-and-the-fsf)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

### 9 responses

1. [Christopher Allan Webber](http://dustycloud.org/) says:[18 August 2014 10:44 PM](#458921ed14a224f7488a914f5c715ab3b950de94)

   Thanks for posting this, Andy. I don't know what to say other than that I hope all GNU projects and members continue to reflect and act in ways to make GNU a more welcoming and safe place for everyone.

2. [Christopher Allan Webber](http://dustycloud.org/) says:[18 August 2014 10:59 PM](#f5b063bd7cc532cfd677e7368316092db85e823b)

   (I say that strongly including myself in GNU members with that responsibility, if it wasn't clear!)

3. Julian Graham says:[18 August 2014 11:11 PM](#2a77038c682160b179748df21dec51dce09dfe23)

   This bothers me! I don't know why I thought GNU would be less vulnerable to this kind of dysfunction, but I did. As part of owning it, let's get the right language back in the policy.

4. Joe Smith says:[19 August 2014 4:45 AM](#bd9a815c99bcb43bbcebab3cf7e28a1341edc5b6)

   Andy, thanks for posting this and I am a long time fan of your work.

   I hope this post doesn't ruffle any feathers....BUT, is it possible that this ultimately exposes a contradiction in the otherwise-lofty neutrality espoused by the OSI guidelines that the GNU licenses conform to? To illustrate...one cannot say offensive things at a GNU conference, but one is completely free to use GNU software to create (among other things) a website which would only be limited by indecency laws? The OSI guidelines indeed seem quite keen to highlight that offense is protected.

   Maybe I am conflating unrelated issues...but at some point, should GNU attempt to reconcile its license guidelines with its conference guidelines?

5. [4](http://4x13.net/) says:[19 August 2014 6:09 AM](#733a5cf14418673ad784e5ad9d1387f3db6d351f)

   Joe, I am pretty sure that the GNU people will not sue you for forking the conference model and making your own free speech symposium.

6. Alex Sassmannshausen says:[19 August 2014 8:24 AM](#a0bedd4721367d498d4c28b4b3300e43d915bb8c)

   Hi Andy,

   Thanks for posting this. I agree with the entire sentiment of the post, so won't add anything to that.

   I did want to mention that I had a different interpretation of the conclusion of the discussion on Sunday.

   I think at the end of the meeting, whilst it was rushed, we had broad consensus to accept the anti-harassment policy with the revised wording.

   I think all 3 changes to the policy were mandated directly by the meeting on Sunday.

   Whilst I was entirely happy with the initial wording of the policy, I'm glad that we now have a collectively agreed upon policy.

   Anyway, thanks again for posting — I think it was definitely important you did so.

   Alex

7. James says:[19 August 2014 10:21 AM](#6054fe5826260f92e18dfb27fccc0d29fd135a3a)

   Are there videos of the GHM 2014 talks available somewhere online?

8. freedeb says:[19 August 2014 7:56 PM](#5a660c54cf9eb953369fa7071b1e07024f429f7f)

   Thanks for posting this Andy! I'm really glad that there are people like you in GNU. The mailing list thread was pretty disheartening, which made asking on list about the in-person conversation unappealing. I'm really glad that there's a widely supported policy in place now.

9. [Mike Gran](http://lonelycactus.com) says:[20 August 2014 4:31 AM](#5776f91e406d4f7c8837454885d1df821781383d)

   In the end, all I have is my own position. I would like to see women in tech. At a minimum, to make that happen, I need to take the action of being inclusive and the self-censorship of not being belittling or inappropriately sexual when in a professional context. This is such a small, easy thing. It doesn't unmake me as a man.

   I watch GNU from the sidelines: a strange and wonderful community.

Comments are closed.
