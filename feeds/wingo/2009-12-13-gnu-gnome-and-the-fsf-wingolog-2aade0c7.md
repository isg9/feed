---
title: gnu, gnome, and the fsf — wingolog
url: https://wingolog.org/archives/2009/12/13/gnu-gnome-and-the-fsf
published: "2009-12-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/12/13/gnu-gnome-and-the-fsf
---

# gnu, gnome, and the fsf — wingolog

## [gnu, gnome, and the fsf](/archives/2009/12/13/gnu-gnome-and-the-fsf)

13 December 2009 9:33 PM

- [gnu](/tags/gnu)
- [gnome](/tags/gnome)
- [free software](/tags/free%20software)
- [fsf](/tags/fsf)
- [ghm](/tags/ghm)
- [rms](/tags/rms)

I have been meaning to write this article for a couple weeks now, ever since the recent [GNU Hackers Meeting](http://www.gnu.org/ghm/2009/) over in Göteborg. I'd rather be hacking tonight, but some other circumstances make such a report a bit more timely; so, here goes!

**GHM**

I arrived a little late, having missed the first day's talks. I was particularly irked to have missed [Bruno Haible's talk on modularity and extensibility](http://www.gnu.org/ghm/2009/haible-extensibility-slides.odp), but I understand that there will be some video up soon.

I didn't know enough at that time to miss having seen José Marchesi's talk, but now that I know the fellow, it is a shame indeed. Actually that was one of my biggest take-aways from that event: that I never really knew GNU as a community of maintainers. GNU in 2009 is not the kind of organization that flies people all over to meet each other, and it's to our loss. I hope to see much more of GNU hackers in the future.

So everyone that goes to conferences knows why they are great: the hallway track. Or the bar track, as it might be. So what were the topics?

**GNU is not FSF**

This point really surprised me: that GNU is not the Free Software Foundation. There a relationship, but they are not the same thing, not by a long shot.

So here's the deal. In the beginning, there was GNU. But Richard realized that in the world of 1985, with proprietary software on all sides, it would be easier to defend the small but growing software commons if hackers collaborated by assigning their copyright to one U.S.-based organization. The FSF was set up as the legal entity associated with GNU, with RMS at its head.

25 years later it's still like that. The copyright assignment paperwork that every GNU contributor signs has some clauses that obligate the FSF to keep their conributions free (in the Free Software sense), but ultimately trust in the FSF is a trust in RMS, and in his principles. It is a testament to RMS's character that there are 35000 lines in `fencepost.gnu.org:/gd/gnuorg/copyright.list`, representing at least 5000 contributors.

But what's up now? As free software became more and more successful, it became clear that there were other ways to get involved than just writing code. There's all kinds of advocacy work that needs doing, for example. The FSF was a natural concentration point for these efforts.

However, also inevitably, with the influx of people, the composition of the FSF changed. There are very few hackers in the FSF now. There is RMS of course, whose work these days doesn't involve programming, but that's about it. Recently [Mako](http://mako.cc) was added, which is an important step to redressing that situation, and more on that later.

I mean, look at [www.gnu.org](http://www.gnu.org/) and [www.fsf.org](http://www.fsf.org/). See the difference?

So the advocacy and campaigning is with the FSF, and the hackers are with GNU. I think every GNU hacker is really down with the message of freedom; I mean, we are the ones hacking the hack. But, as a majority, the GNU hackers are *not* down with "Windows 7 Sins" or "Bad Vista" or most of these negative campaigns the FSF has been running recently.

I say this as a GNU maintainer, not as a representative of GNU, but I believe I have the facts right.

**GNU and RMS**

All GNU hackers respect RMS. We respect his ethical principles, his vision, his tenacity, and his hack. I mean, GNU Emacs: this, for most of us, is the best software in existence. The early days of GCC. Texinfo. Really remarkable contributions, these, without which the world would be a poorer place.

(If you disagree, that's cool; but I'm just trying to explain where we come from. I guess also I should clarify what I mean by GNU here. I mean, "people who have signed paperwork to assign copyright to the FSF". I know that mentioning the FSF there is a bit ironic, but it's a strict definition.)

But nowadays, while RMS's principles remain strong (thankfully), he's a bit absent, technically. That would be OK -- he has certainly put in his hack-time -- but that the GNU project doesn't really have a means to make technical decisions on its own right now. Each maintainer can mostly make decisions about their modules, and we can talk to each other (mostly via gnu-prog-discuss), but there is no governance structure for the GNU project itself.

Worse, sometimes RMS decides things without any input at all, when such input really is needed. The decision to bless Bazaar as the official GNU version control system, for example, really sits poorly with a lot of people. The adoption of SCM and MIT Scheme as two additional Schemes in the GNU project also are real WTF decisions.

These "blessings" don't have much of an effect on people's behavior -- most active GNU modules use git, for example -- but they make people lose faith in GNU's technical coherence.

The issue here is that the GNU project is a community of people working for software freedom, yes, but we have some specific values binding us together. One, the ethical principles of supporting user freedom; more on that later. Secondly, there is a technical vision of a "GNU project" as a cohesive, well-thought-out, integrated technical whole. One can now argue about the extent to which that is currently true, but it is a very important value to GNU hackers.

Things were more or less fine when there was more of UNIX that needed to be reimplemented, and when RMS himself was more on the ball. But now there is a significant measure of dissatisfaction within GNU itself about the way decisions are made. There are some steps being taken to fix this (a recently created GNU advisory board, for example), but it is ironic that some decisions in GNU really do come from one man.

At the same time there is actually a widespread concern within GNU about what would happen if decision making were to be made more open. No one wants outsiders making decisions, it would still have to be maintainers; but there is concern that things might get out of hand, that there might be too many pointless discussions, that bad decisions could be made, that there would be a tyranny of the talkers, etc. This is why things are changing really slowly. Everyone wants more autonomy, but they don't want to lose the solidarity.

**GNU and GNOME**

So what's the deal with GNU and GNOME? Everything, and nothing. Allow me to explain.

GNU people see GNOME as an "outside" thing at this point. To most GNU hackers, GNOME is not quite GNU. GNOME doesn't follow GNU's [coding standards](http://www.gnu.org/prep/standards/) (recall the point about technical integrations), assigns no copyright to the FSF, and seems to prefer LGPLv2+ and not GPLv3+. There is hardly any communication between GNOME people and GNU people. In GNOME, people look at problems their systems are having, and decide to solve them within GNOME -- PolicyKit, for example.

(I don't want to pick on David's excellent software; but I'm equally sure that it never crossed David's mind to suggest PolicyKit for inclusion into the GNU project. There are many similar examples.)

Hell, many GNU people even use things like [Sawfish](http://sawfish.wikia.com/wiki/Main_Page) or [Xmonad](http://xmonad.org/) or [StumpWM](http://www.nongnu.org/stumpwm/) or other such things. Admittedly, GNU folks would be more likely to install GNOME on their cousin's computer; but as for themselves, Emacs is the only program many people need.

GNOME people on the other hand are in two camps, more or less. Broadly speaking these camps correspond to Free Software and to Open Source, respectively. The former feel a strong heart-tie with the GNU project; perhaps they started working on GNOME back when it really was a GNU project, or perhaps they started with it because they believed in GNU's ethical principles and chose that environment because they wanted to spread user freedom to their friends. As I say, most GNOME people are disconnected from GNU per se, so this tie really is more cultural than practical, but it is there, and it is strong in its own way.

The Open Source people tend to value GNOME in particular from the days when Qt was GPL-only, or even proprietary, and GNOME allowed them to develop open source applications as well as proprietary ones. Open Source people appreciate the technical comraderie, as well as the technical excellence of the platform, but often disagree with the GNU project's copyleft principles, instead appealing to individual choice as to whether to use or make proprietary software or not.

I hope I haven't been uncharitable to anyone here. Please correct me in the comments if so.

**is GNOME GNU?**

So, Andre [posts, wondering about the relationship between GNU and GNOME](http://blogs.gnome.org/aklapper/2009/12/13/to-gnu-or-not-to-gnu/). I hope I have been able to add some broader context to the question.

I think that Andre's characterization of RMS as "fascistic" is totally wrong. There are serious problems of decision-making within the GNU project, yes, but "fascistic" takes it a bit too far, and almost brushes Godwin's Law ;)

The particular context in which the discussion has been brought up, that nasty thread on foundation-list, is unlikely to bring about any mutual understanding. It is ironic that the topic is the code of conduct, which was designed to promote understanding.

Bottom line, GNOME can be GNU if it wants to. But I don't think a decision is necessary right now, and certainly not on foundation-list, not in that thread of the talkers.

**GHM redux**

I promised to talk more about hallway tracks at the GHM, but I've run out of semicolons. Until next time!

## related articles

- [here we go again](/archives/2021/03/25/here-we-go-again)
- [state of the gnunion 2020](/archives/2020/02/09/state-of-the-gnunion-2020)
- [on gnu and on hackers](/archives/2014/08/18/on-gnu-and-on-hackers)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [thoughts on rms and gnu](/archives/2019/10/08/thoughts-on-rms-and-gnu)

### 23 responses

1. foo says:[13 December 2009 10:18 PM](#c6d86fa03a25bef65edaf57586594a2343ae2170)

   Argh. The GNOME people seems too inclined to name calling. Every time is the same. They really like drama and conflict as well.

   I liked the honest that you put on behalf of the relation between the GNU project, FSF and the decision making. This seems an issue to be dressed and more important, that \*can\* be dressed.

   If the GNU and GNOME project are not that close, as you said, let them go. They seems each day more apart from the Free Software ideals anyway and, as I mention, too inclined to name calling, which is really not cool.

   The GNU project could choose a WM which reflect better its own principles. That would be cool.

2. Simon says:[13 December 2009 10:37 PM](#e3394e673b4cdb77327ba69dbff3934ac6746740)

   I honestly don't think there's much connection between Gnome and GNU these days. RMS talks about the reasons for starting the Gnome project, but that's ancient history to most current developers, and I don't think many of them see much relevance in it.

   As I see it, most developers are driven by different motivations from the FSF - be it technical challenge, or the desire to contribute something. They're generally more pragmatic about proprietary software, recognizing that users are attracted to quality software, not to philosophical statements - if a commercial product is good, it should be held up a good example, not condemned as proprietary trash.

3. Simon says:[13 December 2009 10:45 PM](#ca96b5951e54b583f07e27be61950b6859352c25)

   @foo - I don't see a lot of name calling happening here. It's a simple, and mostly civil disagreement - RMS strongly pushing the FSF message, unhappy with what he sees as promotion of proprietary software by Gnome. And Gnome developers disputing a) that they shouldn't be allowed to speak positively of non-free software, and b) that personal blogs syndicated on Planet Gnome are in any way a statement on behalf of Gnome itself.

4. Markus says:[13 December 2009 11:59 PM](#54551d3c3dd8ba8b1e7eee53b2a644a909bfe8c5)

   For me GNOME stopped to be a GNU project when Sun and HP founded the GNOME Foundation.

   GNU is a community project. There are corporate employees working within that community, but it's not a "corporate-led community" (like, for example, OpenOffice or various Linux distributions).

   GNOME started as a project within the GNU community, but ever since the rival GNOME Foundation was founded by corporations, the GNOME community put their matters in corporate hands.

   Surprisingly, KDE learned from their early mis-steps (non-free toolkit), while GNOME moves farther and farther away from their own former agenda (which was "be more free than KDE") to a point where even an official split is considered to defend De Icaza's drooling over Microsoft's proprietary technologies like Silverlight.

5. foo says:[14 December 2009 1:35 AM](#a4b5f9629cf7e062a67564ea7ec0e8e064d3d766)

   @Simon If you meant "here" as in "at this blog post", yes, I certainly agree with you. I really liked this one.

   The blog posts I was speaking about is the other ones at the Planet. And not just the ones specifics about this issue. All the community, or most of it, or the most vocal of them, choose what you want, are always ready for a fight. And that is not healthy.

   And see, I am not saying that one have to agree with everything nor nothing. I am saying that the ones who do it always do it in a very aggressive way or bend the facts to make a good argument point against it. That's not cool. That's not healthy.

6. foo says:[14 December 2009 1:48 AM](#028e22a27f22cde605cf8001354e8ca55bc3231f)

   And I think this is obvious, but nevertheless: the FSF is \*against\* closed proprietary software. Period. One can argue that one is not, but the FSF \*is\*. And if you are a GNU project, or any project really, the least thing to expect from them is complain when you promote a proprietary software.

   The people who happily shout "We want good software. We don't care what" are not thinking really hard, I think.

   See, using this logic, no freesofware would ever been made. The initial ones, and a lot today, are made because someone cares about choice. Every one who writes software knows how crappy it is until it archive a certain level of "suck less". I find the just-quality-matters-dance troubling and very dangerous. Yes, quality matters and everyone should pursue it. But just as such.

7. Hatem says:[14 December 2009 3:23 AM](#1480b61edd6c48816636a9e055311bb65fe4ced3)

   @foo: What bothers me though, is that the gnome \*project\* is not promoting proprietary software by any stretch of the imagination.

   The content in question was posted on a blog aggregated on planet gnome, which contains this very clear and simple disclaimer:

   “Planet GNOME automatically reposts blog entries from the GNOME community. Entries on this page are owned by their authors. We do not edit, \*endorse\* or vouch for the contents of individual posts.”

   (emphasis added, from http://planet.gnome.org/#footer)

   The planet is very obviously not an official mouthpiece for the gnome project. Hence I don't see why it is such a big deal if an author discusses proprietary software favorably on their blog.

8. [James](http://jamesgecko.com) says:[14 December 2009 4:00 AM](#b049080c9508c81db054c4667136c1d7e8df0595)

   "See, using this logic, no freesofware would ever been made."

   @foo: Really, now? I think you're just making stuff up. There's plenty of examples of free software that was made because there wasn't \*any\* implementation. Similarly, implementing stuff \*just\* for the sake of choice is lame because then all we'll have are clones of pre-existing software.

   There's a difference between mentioning/discussing proprietary software and promoting it. Stallman seems to think it's all one and the same.

9. foo says:[14 December 2009 4:29 AM](#1e4688381ba65176f19d4b6f056f9ec0fa2a5c1a)

   @Hatem Yes, I suppose you're right about the Planet not being "an official mouthpiece for the gnome project" and I personally don't see either what a big deal about discussing proprietary software on it.

   On the other hand, the GNOME project is not a supernatural entity that exists without people. The people working and using it are the GNOME project. And, if the people who build it discuss proprietary software, to me it says something about the values pursed by the GNOME project itself. (I am definitely not putting any judgment of value, good or bad, here.)

   @James I think you're not being honest with what I said. I too can come up with infinite situations where software were make that will not add anything at all to the discussion. Please, don't do that.

   And to respond your last paragraph, I do too believe there's a difference, although I am afraid that I don't agree with the distance between them. If you say good things about it, you are in a way promoting it.

10. James Henstridge says:[14 December 2009 8:04 AM](#8a1923fc50b39f5d51edcf731cd18bd83ca9dd83)

    @Markus: that's a pretty severe mischaracterisation of the process that led to the creation of the Gnome Foundation.

    I was on the Gnome Steering Committee that was formed at the first GUADEC conference that was a predecessor to the Foundation board. If there was one primary reason to create the foundation, it was to manage finances.

    The project had won a sum of money in an award (I can't remember what it was exactly -- I could look it up if necessary). There was no legal entity to pay the money to, so it was transferred to the FSF on our behalf. The FSF then wanted oversight over how we spent the money, and at one point wanted to treat it as general funds rather than being earmarked for use by Gnome.

    Creating the foundation was a way to get control of those funds, and make sure we could handle future donations/prizes in a sane way.

    As far as the "corporate-led" accusation, I'd suggest you read the bylaws some time. In particular, the sections about who elects the board and limits on the influence of any one company. The community certainly has more of a say in how the Gnome Foundation is run than e.g. the FSF.

11. [Peter](http://sjamaan.ath.cx) says:[14 December 2009 9:39 AM](#4689582a6dfe60e95606dff88b32bf2c122c1c51)

    What about GNUStep? It even has GNU in the name, and I always was under the impression \_that\_ was the "official" GNU desktop environment.

12. Markus says:[14 December 2009 9:55 AM](#de8b60a2bfe11c879e93d980c3a2ca7d764f5449)

    James wrote: "The FSF then wanted oversight over how we spent the money (...) Creating the foundation was a way to get control of those funds"

    Thanks for explaining that it was about money. That doesn't change my opinion that the GNOME Foundation marked the end of being a GNU project. Now I just know the reasoning behind its founding.

    "As far as the "corporate-led" accusation (...) The community certainly has more of a say in how the Gnome Foundation is run than e.g. the FSF."

    Quote from http://linuxdevcenter.com/pub/a/linux/2000/08/18/rt.html:

    "his consortium of several traditional IT companies, like IBM, Hewlett-Packard and Sun Microsystems, as well as Linux companies like Helix Code, Red Hat and Eazel"

    Doesn't sound like a community project. And when you write that the FSF has only a minority vote in the Foundation, it confirms my opinion that GNOME stopped bein a GNU project back then.

13. Stelios says:[14 December 2009 10:03 AM](#842853670bb8e8115b494e377e1ae5fa157ad730)

    That was a refreshingly honest and informative post. I wish there were more people like you in the Free Software World; ready to illuminate rather than obliterate.

14. pinky says:[14 December 2009 10:43 AM](#7b723d83a1a7131aa43ca60afc6eddaba59625ad)

    @James Henstridge: Thanks for explaining the reasons behind the GNOME Foundation. Nevertheless I think it's a shame that this kind of problems (earmarking of funds) couldn't be solved without creating an own foundation.

    Generally GNU projects seems to have very little in common these days. No central place for web presence (some projects are at gnu.org/software/...) but not all. Not every GNU project does copyright assignment, no common development place (not every GNU project is at savannah),... No common developer conference (GNOME has Gudadec, than there is GHM where not that much hackers attend, described by Andy Wingo) etc. And some projects even have their own Foundation.

    Also what I, as a user, don't like are the license decisions in the past. E.g. Evolution switched from GPL to LGPL to allow non-free plugins. What the f\*\*\*! \_The\_ PIM client of the GNU projects choose a license with non-free software in mind?

    I want to thank Andy Wingo for his blog article. It shows that some GNU hacker have the same feeling about GNOME that I had.

    On the other hand, like mentioned by Markus, I'm positively suprised by KDE. They seems to come more and more closely to the free software movement. Running fredom-related campaign. Like the "Be-Free" slogan since KDE4. Pushed KDE strongly toward (L)GPLv3. Take care about legal maintainance by adopting the FLA of FSFE, etc. Sometimes I feel like KDE is the secret GNU desktop. For me, as someone who loves freedom, the GNU \_and\_ GNOME that's really sad.

15. Kevin Krammer says:[14 December 2009 10:49 AM](#f14385ae6ee88caa0c795f5d0c1096fdb481aa8f)

    Very insightful posting!

    You've only got a little error in there: Qt was never "GPL-only", it always allowed prorietary software development.

    (I'll leave it to everyones personal point of view whether allowing that is a good or bad thing)

16. James Henstridge says:[14 December 2009 11:28 AM](#eee7de37bd0c0e4b155d0eba30301202593fd91e)

    @Markus: pretty much every non-profit corporation comes down to managing money or other assets. The FSF is the same when you get right down to it.

    We would have been happy to continue using FSF as the legal entity managing the funds if they had agreed to treat funds earmarked for Gnome separate from general funds.

    As far as the management of the Foundation goes, I again suggest you look at the bylaws:

    http://foundation.gnome.org/about/bylaws.pdf

    The foundation is controlled by its board of directors. The board of directors is elected by and accountable to the foundation members. The foundation membership is made up of contributing members of the community (i.e. not the companies on the advisory board). There are limits on the number of directors who can be affiliated with any one company.

    This is a lot more control than GNU contributors can exert on the FSF.

17. Markus says:[14 December 2009 1:18 PM](#072ec9093632195383335f428eb8ec31f352aca8)

    I never wrote that creating the GNOME Foundation was a bad thing. I wrote that IMO the Foundation was a de facto split from GNU.

    So all this talk about leaving GNU is nonsense, because you already left GNU nine years ago. Go ahead and leave GNU officially. It won't make any difference. Someone needs to delete "is part of the GNU Project," from http://foundation.gnome.org/, but that's it. GNOME, the Foundation, and the FSF will operate business as usual on the next day.

    If you leave GNU, at least you'll finally be honest with the world, as for the entire decade the "we are GNU" formality was a de facto lie.

18. [andre klapper](http://blogs.gnome.org/aklapper) says:[14 December 2009 7:00 PM](#443d370115073552105b15222b857cb60d192978)

    @pinky: "Evolution switched from GPL to LGPL to allow non-free plugins. What the f\*\*\*!"

    This happened as Evolution needed to link against Samba4/libmapi (GPLv3) to provide access to recent Exchange servers (like 2007).

    It's a fact that Exchange exists, plus its users that were otherwise unable to use free software to access Exchange servers.

19. pinky says:[14 December 2009 7:09 PM](#ad9fa6f8cbe947b279a7bad524bf910fc69d40c7)

    @andre klapper: I remember a blog post form an Evolution hacker which explicitly mention non-free plugins as an motivation to switch to LGPL.

    If it was only Samba and the GPLv3, than why Evolution doesn't simply switch to GPLv3 or at least use GPLv2+ (which would be GPLv3-compatible).

20. pinky says:[14 December 2009 10:18 PM](#5a8c954af68fdcee3c52411ee02199efbc346d00)

    btw. just an idea. If GNOME decides to stay in the GNU project (I would appreciate such a decision). What about a joint GUADEC and GHM? The last GUADEC was together with Akademy but I think it would be good and about time that GNOME and GNU come more closely together and get to know each other and to understand each other again.

21. anon says:[15 December 2009 0:55 AM](#a641dc2659e73ee2e8d126ab36bd40a21e7a64bc)

    really informative post, thanks!

22. [Sweetandy](http://www.sweetandy.net/) says:[12 April 2010 11:30 PM](#4859d7fbb05657f21cf25e53c8ae44b91357be8b)

    Somebody linked to a blog post from wingolog in the GNU Social IRC channel, and I stumbled across this post. It provided a rather comprehensive overview of exactly what I had been wondering about the relationship between GNU, FSF, and GNOME for years now. Thanks, Wingo!

23. obneq says:[15 April 2010 11:32 AM](#fddc016a3280a68d88118f82eefc13a8b4f3ff55)

    im very much with peter here: give GNUstep a good look!

Comments are closed.
