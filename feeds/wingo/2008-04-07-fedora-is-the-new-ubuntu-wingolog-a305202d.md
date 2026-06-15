---
title: fedora is the new ubuntu — wingolog
url: https://wingolog.org/archives/2008/04/07/fedora-is-the-new-ubuntu
published: "2008-04-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/07/fedora-is-the-new-ubuntu
---

# fedora is the new ubuntu — wingolog

## [fedora is the new ubuntu](/archives/2008/04/07/fedora-is-the-new-ubuntu)

7 April 2008 2:20 PM

- [fedora](/tags/fedora)
- [ubuntu](/tags/ubuntu)
- [nargery](/tags/nargery)
- [flamebait](/tags/flamebait)

In the last few months I have come into custody of a few new machines for work. I decided to install [Fedora](http://fedoraproject.org/), first on my workstation (an 8-core monster) and then on the laptop (an all-Intel Thinkpad X61), for ideological and social reasons. It's like this: Fedora is the new Ubuntu.

Ubuntu was great in its early days, because it took upstream to the people, bringing the 6-month GNOME release cycle into users' hands quickly and predictably. But over the past year or two Ubuntu has become less and less interesting. Part of it, I feel, is that Ubuntu strays too far from free software. Ubuntu is willing to accept and accomodate proprietary software in the distribution. In a related fashion, resources that go into Ubuntu/Canonical do not, in a large part, go to further the development of free software, because most development that Canonical pays for goes into the downstream product "Ubuntu" and not into the larger free software world of the kernel, glibc, GTK+, GNOME, etc.

This admittedly broad-stroked characterization stands in marked contrast with Fedora/Red Hat. Fedora is the distro used by most people that are hacking on the parts of the free software desktop that are important to me and my machines: the people hacking X and the intel drivers (Anholt *\-\- ed: works for Intel, hacks FreeBSD*, Jackson, Airlie), the people hacking all the interesting hardware integration (Zeuthen, Hughes, Larsson), the people hacking the core GNU toolchain (Drepper), the people maintaining core GNOME libraries (Clasen, Worth, Esfahbod), etc. And, of course, the [kernel](https://www.linux-foundation.org/publications/linuxkerneldevelopment.php).

Furthermore, Fedora finally seems to be turning into a real community project, rather than a simple testing-ground for RHEL. The unification of Core and Extras is the most visible sign of this change. Fedora's where it's at.

Anyway, with my recent laptop install, I'm in week three of the new all-Fedora world. It's been OK to me thus far, once I got over the culture shock. Yes, yum is still much slower than apt, although it has gotten better recently, and rpm's incredibly non-orthogonal interface is just beyond my understanding. And of course I turned off SELinux entirely on the laptop. I don't think I'll be moving over my servers -- Debian is too perfect -- but for a desktop machine, Fedora rides close to upstream development, which keeps things interesting.

Flame on!

## related articles

- [/usr/local, fedora, rpath, foo](/archives/2011/03/18/usrlocal-fedora-rpath-foo)
- [so you want to build a compositor](/archives/2008/07/26/so-you-want-to-build-a-compositor)
- [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)
- [hear, memory](/archives/2008/04/08/hear-memory)

### 96 responses

1. [king.pest](http://king.pestarium.com) says:[7 April 2008 3:06 PM](#38eed2a9ed69d7fa2cabdb58830700df9487a74c)

   hey, but how about overall performance? is it because of your 8-core monster or because of some new Fedora patches? I just can't forget how slow Fedora was, and that was its biggest disadvantage for me, especially comparing it to a quite fast Ubuntu.

2. [Mårten Woxberg](http://shifthappen.blogspot.com) says:[7 April 2008 3:16 PM](#aca09427461078ac900a78ef7462f215a7aa678a)

   As I'm trying to spread Linux(Ubuntu) amongst my friends I prefer that it works on their hardware without glitches, if it takes proprietary drivers for that to happen right now, so be it.

   I'm never ever going back to RPM hell...

3. Anon says:[7 April 2008 3:20 PM](#4c3e0baa622ce725637e63b8ae0ffafddf180d61)

   What about all the folks who work at SUSE (e.g. Michael Meeks) who do GNOME and OpenOffice desktop work? Don't X hackers like KeithP use Debian?

   Finally, I wish people wouldn't turn SELinux off, we need it to be fixed not abandoned.

4. [James Henstridge](http://blogs.gnome.org/jamesh/) says:[7 April 2008 3:42 PM](#35f4b71d5c51557591c3c8e7866373e9b9708536)

   Note that Canonical is at least an order of magnitude smaller than Red Hat and Novell, so you'd expect to see more R&D work from them.

   That isn't to say that Canonical doesn't give back anything to the community: a lot of the focus has been on tools to make developers more productive. Bazaar and Launchpad are the primary examples of this.

   \[as a side note, your blog software doesn't seem to like posts with ampersands in them\]

5. [Brit Butler](http://www.redlinernotes.com/blog/) says:[7 April 2008 3:46 PM](#c330110bc2c796d4cf5d46d8b7f158064ca3549a)

   I'm in the same boat as king.pest. Each distro release cycle I test the betas on Fedora and Ubuntu. Sometimes I install both. I've always felt like Fedora is needlessly slower than Ubuntu and been unsure of the cause. It's not just yum either which does, IMHO, pale in comparison to apt-get. Finally, I never get around to adding repos to put mp3-codecs on the distribution. Now, the speeds the breaking point and I'm happy to work around the other issues. I'm also happy to support a distro that, as you said, is more upstream-focused than Ubuntu (though I think paying them to work on launchpad and try to tie things together further down is an interesting strategy). I just shouldn't need a bulky system to get done the simple things I want to do.

6. [Mike McGrath](http://mmcgrath.livejournal.com/) says:[7 April 2008 3:46 PM](#f4f2ce85cbeea9cb812f64b5ceb0a0ca47ac6447)

   Also Fedora has the backing of Red Hat, a profitable enterprise. Fedora's not going anywhere. On the other hand Canonical is still mostly completely reliant on Mark to pour money into it. That money (marks money) will run out eventually and if its before Canonical turns a profit, Ubuntu's future is questionable.

7. jef says:[7 April 2008 3:46 PM](#6277cca25a9c8b9eb66f6d7692ed832c0a6163e4)

   James, How is a Canonical controlled proprietary service giving back to the community? How many Canonical developers are working on the proprietary service as opposed to open code? How many new upstream projects has Canonical initiated?

8. janonymous says:[7 April 2008 3:49 PM](#f4783e1773080d78c346d9b45154bd83bbcca2e1)

   You are a fucking communist! When will you pinkos learn?

9. [Mathias Hasselmann](http://taschenorakel.de/mathias/) says:[7 April 2008 3:54 PM](#8ce7177ec03060b206f3377852d3b79c13b2cc53)

   Well, I consider trying Fedora from time to time - until I remember the pain of RPM. Debian packages are just too easy to create and patch. Not to forget: Aptitude.

10. loeppel says:[7 April 2008 3:59 PM](#e5a9977f48049b43fe438b033d59202e8b8c4e60)

    jef

    upstart for example - which is now adapted to debian and there is a fedora release too.

    You mentioned that Fedora is the new Ubuntu, but why?

    Because the Fedora people has copied much of Ubuntus ideas!

    Fedora in the ealry days was a unusable test distro (Core 1 and 2). Nothing to compare with RedHat 9 (!) or SuSE (not openSuSE) 9.x.

    And the openSuSE project from Novell, is also caused by the "Ubuntu hype" - no one will buy a SuSE Box when he can get the same for free.

    Sorry for my bad english.

    greets,

    loeppel

11. bronson says:[7 April 2008 4:05 PM](#e25b65127bbb9313e15d557208ef367cfc9c0ffe)

    jamesh, Launchpad is a perfect example of what this post is talking about. What do you think a closed-source, closed-data, underperforming web service demonstrate about Canonical's commitment to the community? I'm very curious to hear your reply.

12. bla says:[7 April 2008 4:11 PM](#6f69df89c8e0ddab35f7c370e612e5ab99fa21c6)

    jef: canonical doesn't pay for the development of upstart. It's just a pet project Scott pursues in his free time.

    Canonical doesn't do development. At least not of relevant Free Software. Yes, they do minor patches here and there. But last time I looked their greatest contribution to upstream Free Software was a redesign of the GNOME logout dialog. Yay!

    (Yepp, they develop Bazaar. But quite frankly: Bazaar is nothing more than yet another implementation of an already known idea. And not even a particularly good one. It's slow. And nobody uses it. I hope it stays that way. The balkanization of the VCS world is already advanced too far.)

    Damsn freeloaders! ;-)

13. jef says:[7 April 2008 4:15 PM](#dec2db387d80bc46001a1f252be53570345b4062)

    loeppel,

    upstart is lonely example. Look at Red Hat contributions in contrast

    http://fedoraproject.org/wiki/RedHatContributions

    Moreover there is nothing proprietary in Fedora. Koji, Bodhi everything is free and the entire repository is open to everyone. You say Fedora copied which is a good laugh.

    For those who bitch about RPM hell and comparing to Apt, you haven't learned much. First of all, there is Apt in Fedora too and works fine with along yum and smart and uses the same repository.

    http://fedoraproject.org/wiki/Tools/Apt

    Yum is really fast in Rawhide too.

14. [wingo](http://wingolog.org) says:[7 April 2008 4:20 PM](#9a20069de2b87691e15e9415456642e26389ad87)

    Random thoughts before hopping on a train: the thesis is not that "other distros don't do anything for free software", it's that fedora does a lot, and is in a very sweet spot.

    Also: the assertion that Ubuntu/Canonical prizes differentiation over free software.

    I feel no need for SELinux castigation on a humble laptop used for development. On a server, maybe.

    Let the record also show that I am fond of Bazaar.

    And regarding the humble ampersand: this form thing accepts xhtml, with minor concessions to making paragraphs. Ampersands are syntax in xhtml, and thus need escapage. Perhaps I should implement a non-xhtml parser though.

15. miscz says:[7 April 2008 4:29 PM](#c600d751f3c8f9874ff5f3265d2e1a45f8c3b09a)

    So what if Ubuntu doesn't do much upstream work? The real downside is that some projects are integrated into it slower than in Fedora (like Network Manager). Canonical money goes into creating a good distribution and they're good at this IMHO.

    That's what I look for in a distro:

    -good package management (apt/dpkg are just fast, Synaptic is both more easy to use and more powerful than Fedora 8 package manager GUI)

    -big repositories (Debian folks do amazing job on that)

    -wide range and high quality of third-party repositories (ever added few repositories in Fedora and get a huge clusterf\*ck of questions, dependency problems when trying to install something?)

    -fairly recent versions of software (but not to the point where I test unstable release)

    -no unnecessary fuss with closed-source stuff - if I don't want it I won't install it but not including it in repositories is silly when you aim for mainstream use

    flame on :)

16. [ulrik](http://users.student.lth.se/f04us/) says:[7 April 2008 4:42 PM](#db52af1ad393455cc2cf72af56b294c316f75794)

    Red Hat should buy out Scott J.R. and pay him for working on Upstart as a cross-distribution intelligent init. That would benefit ubuntu, fedora and all linux distributions..

17. apokryphos says:[7 April 2008 4:44 PM](#9f375b0a5bed7e825fa34476899ca36691e64d43)

    Don't forget Novell/SUSE in there. They actually employ a lot more Linux \_desktop\_ developers than Red Hat for example, and have many kernel, X, GCC, etc. developers ensuring the growth of Linux on the desktop as well as the server as well.

    All in all, they have a few hundred developers writing code on different parts of Linux. It's sad that you didn't give them acknowledgment too in your blog post.

18. jef says:[7 April 2008 4:49 PM](#a858ae73225ad5afc1ba05403f479c2ec221917d)

    "Don't forget Novell/SUSE in there. They actually employ a lot more Linux \_desktop\_ developers than Red Hat "

    Really? Where do you get your numbers from? References? Red Hat maintains more GNOME and freedesktop components than anybody else. Just look at the wiki page referenced above for details.

19. frits says:[7 April 2008 5:16 PM](#f22431d17d9ceeb584e33dabace389a9b42de8f5)

    Fedora's almost dogmatic approach towards free software will prevent Fedora from being the next Ubuntu. That said, it does not stop Fedora from making a superb desktop distribution. And if the purpose of your post was to make clear that Fedora (and RedHat) deserves far more credit than it is getting these days, then i couldn't agree more.

    (Exaggerating:) If it weren't for Ubuntu, linux wouldn't be that popular. If it weren't for Red Hat, linux wouldn't be.

    From time to time i try Ubuntu, but i never see any compelling reason to switch (there aren't that many differences, so i just stick to what i am used to).

    One point of critique: i must agree with the few comments here that Fedora (not just yum) seems a bit slower than Ubuntu. Should that be reported in bugzilla?

20. Joe Buck says:[7 April 2008 5:23 PM](#e32d7049c924ced6bfba24252d6586329f0de159)

    frits: users can easily run Fedora+Livna, to get the advantages of Fedora together with the most commonly needed patent-encumbered components. Red Hat people cannot tell you to do that, to protect the company from liability.

    As for questions of whether Fedora is slower than Ubuntu: if someone can create a reproducible test that shows this, by all means file the bug. But a vague "Fedora seems a bit slower than Ubuntu" is a worthless bug report; what could be done with it?

21. Joe Buck says:[7 April 2008 5:27 PM](#9b8bc8c4ef45b6e4c6cd2be6caca8f56d102f201)

    Oh, and "RPM hell" no longer exists. While apt is faster than yum, yum effectively works in the same way, resolving dependencies so things just work.

    Even if you get an unofficial RPM that someone built outside the regular process, installing with "yum localinstall" will resolve the dependencies, as long as it's compatible with your distro.

22. Ben williams says:[7 April 2008 5:40 PM](#7306a2c093c529b3284b4e42cf156a73e64a20c2)

    now with all this talk of fedora and ubuntu for those who want to try Fedora. Fedoro Unity Project released a new re-spin this week

    http://fedoraunity.org/fedora-8-re-spin-20080331-released

    take a look and give it a spin

    (yes we use jigdo) :)

23. [daniels](http://www.fooishbar.org) says:[7 April 2008 6:09 PM](#7ad72bdad236560c88f1a608524f78b2f395627b)

    While I completely agree, and definitely don't want to disparage any of Red Hat's great work on X in particular, Eric Anholt works for Intel, rather than Red Hat.

    Joe Buck: RPM hell comes from a number of things; a packaging system which can only be described as 'shithouse' being merely one of them. It's the failure to package everything under the sun, necessitating sites like livna and rpmfind; the lack of tools like Debian's policy, et al. Debian rises above the usual RPM hell for a number of reasons, of which dpkg and apt are merely two. Why others stick with rpm and yum honestly (honestly) baffles me, after having tried to use both; much as I'm somewhat blinded by having previously worked on Debian and Ubuntu for years, I don't think this is mere parochialism.

24. anon man says:[7 April 2008 6:13 PM](#9cbc5d43b43c2ded5a589e7c8a3d864aea4c563a)

    fedora will be the new ubuntu when RH stops being an idiot and they drop the half assed RPM and adopt DEB and all the good tools like the real apt/aptitude

25. [Mike McGrath](http://mmcgrath.livejournal.com/) says:[7 April 2008 6:18 PM](#7bc64881857cf9c2f571e39a5b3ffaabd2582461)

    Wake up guys. Its been YEARS since I've been in "dep hell"

26. frits says:[7 April 2008 6:44 PM](#9584b9273bed9b09c222ee220cf266f22267bc6a)

    @Joe:

    Don't worry, i wouldn't dare to file a bug stating that "Fedora seems slower than Ubuntu". Of course that would be too vague. However, it is something i notice when trying Ubuntu, and i do wonder how that can be.

    (In case it was not clear, i am a Fedora user.)

27. jefj says:[7 April 2008 6:48 PM](#907f1a3f9762d0c9bf8c27b8b48d3237a9d5a81e)

    "Joe Buck: RPM hell comes from a number of things; a packaging system which can only be described as 'shithouse' being merely one of them. It's the failure to package everything under the sun,"

    WTF are you talking about? Fedora has some of the best packaging guidelines

    http://fedoraproject.org/wiki/Packaging/Guidelines

    And No, it is not a "failure" to package. It is a deliberate decision only to package patent unencumbered free and open source software. Livna is full of Fedora contributions who compliment the official repository. They work together just fine.

    Hey anonman, which century are you living in. Apt has been available in Fedora years.

28. jef says:[7 April 2008 6:49 PM](#a74bb0159208bee785b487e8cf0fa31e2b58313a)

    frits,

    I highly recommend checking out the yum in Rawhide (fedora development branch). It is much more faster. Seriously.

29. Greg says:[7 April 2008 7:03 PM](#7ad4c3bfb5c0fcfd42cdc73c3d79230b92d9c324)

    "Fedora is the new ubuntu" come on Fedora can't possibly be that bad.

30. frits says:[7 April 2008 7:41 PM](#224191ffbc574d5709b0d21c7097f493d5795aec)

    @Jef:

    I don't really have complaints about yum; yum has improved a lot over the last few releases, and besides that, it is not like i use that several times a day anyway. Of course, i look forward to the upcoming F9 release.

    My point was about the general performance of Fedora. Let me be clear, Fedora is not slow, it is by all means fast enough for me. However, like the king.pest in comment #1 and Brit Buttler in #5, i too noticed that in general, Fedora is slower and less responsive than Ubuntu.

31. jef spaleta says:[7 April 2008 8:31 PM](#60b32719b37ff3d351babb3bbc0e58f21f581277)

    Hey look another one 'f' jef.

    I don't have anything to say.. I'm still too hung over from four days of curling. But since someone already gave be credit for the 'Jef' postings here in outside communications I just want to go on the record that I'm not the Jef you are looking for.

    Though I will admit that the post so far incorrectly ascribed to me are eerily similar to my own opinions and I can see how some other people so far commenting would be fooled into thinking its me. Suckers.

    -jef"needs to change his name to a trademarked symbol and introduce his own UTF-8 font encoding which renders the symbol correctly, so that no one has to be confused like this ever again."spaleta

32. jef spaleta says:[7 April 2008 8:33 PM](#c2ef887e6ca1eb539f6f8bf1a5760dce858d094f)

    Hey look another one 'f' jef.

    I don't have anything to say.. I'm still too hung over from four days of curling. But since someone already gave be credit for the 'Jef' postings here in outside communications I just want to go on the record that I'm not the Jef you are looking for.

    Though I will admit that the post so far incorrectly ascribed to me are eerily similar to my own opinions and I can see how some other people so far commenting would be fooled into thinking its me. Suckers.

    -jef"needs to change his name to a trademarked symbol and introduce his own UTF-8 font encoding which renders the symbol correctly, so that no one has to be confused like this ever again."spaleta

33. Stan says:[7 April 2008 9:54 PM](#600240b98c8ad324d7abbc117b9553726fadd575)

    Red Hat and Fedora are currently among the worst distros when it comes to supporting KDE. There is no way to have a KDE desktop by itself in Fedora because all the system tools are built on GNOME -- look how the installer sticks out like a sore thumb within KDE. If Red Hat devs have to lift a finger to accommodate KDE, there's only one answer you'll get from them: too bad, we won't do it.

    This anti-KDE policy stems from a corporate decision Red Hat has made long ago to standardize on LGPL-licensed GTK so that ISVs can make binary blobs without giving anything back. That's why if you're siding with free and open source software, KDE is the desktop for you.

    Kubuntu has done a better job making sure that all the needed system utilities are built using Qt/KDE so that no GNOME components are needed on a Kubuntu installation.

34. annonymous says:[7 April 2008 10:35 PM](#c6657068e9af5effa45d93bce121a87cba3cad9e)

    Fedora != Ubuntu and never will. Let's face it different goals for each one. Plus I do not see Ubuntu not been interesting. Seems like with every release they bring something new to the table.

    A three letter acronym why I stay away from Fedora: RPM

    or

    should I say: RIP

    I will switch to Fedora when they have all those packages that Debian carries plus an easier package manager, plus a good ubuntuforums.

    Seems like ubuntu has a lot now that Fedora/Suse/\[feel in the blakn\] wish to have.

35. [apokryphos](http://francis.giannaros.org) says:[7 April 2008 11:27 PM](#d62d8358cf4082112301a4af53025e4871354a52)

    @jef:

    General inspection. Apart from SUSE's GNOME developers (not sure about a direct comparison with RH, would be interesting -- at least half of the "co-maintained" on that RH list there are with another SUSE employee though), they still employ more KDE developers than any other distribution (I don't think RH employ a single person to work directly upstream on KDE, do they?), second largest contributors to OpenOffice.org (something like 15 developers there alone), X (including all the new RadeonHD drivers), Compiz, ALSA, etc.

    So RH definitely do make a larger contribution to the kernel than SUSE (although, note SUSE's 200% increase in contribution there), but it seems like SUSE are pushing the desktop slightly more.

    Not to put down Red Hat here at all, really -- obviously they're an awesome company doing great things. I just think SUSE should have got a mention too.

    @daniels:

    most of what you're saying is either FUD or completely out-dated. From a technical stand-point it's hard to see many real advantages to dpkg.

    \\* RPM has had bi-arch compatibility for years.

    \\* "Packaging everything under the sun" -- the openSUSE build service has something like 30,000 packages in it these days. We also have a lot \_more\_ backports (like the latest version of KDE, GNOME, X, etc) for all older (i.e. non-EOL) distributions, so users are not forced as much to always have the latest version of the distro installed.

    \\* APT is fast, but this is not any bad reflection on RPM itself. For example, in openSUSE's current development version package management uses a new SAT solver which is actually very slightly faster than APT, and it's a \_smarter\_ solver. It's also more powerful, like local/remote RPM installation, 1-click-install support, etc.

    DPKG and APT were good, but they're not exactly evolving or getting better -- RPM, ZYpp, etc are.

36. jef says:[7 April 2008 11:50 PM](#0b72aee5182478c3e1b3213e799018b5d7283c83)

    apokryphos,

    Your general introspections just looks like guesses without any actual numbers in place. Unless you have solid numbers, the claim that Red Hat has less desktop people than SUSE seems odd considering the fact that Red Hat maintains many freedesktop.org technologies and GNOME stuff like HAL, DBUS, Cairo, NetworkManager, GTK, Nautilus, GVFS/GIO etc and yes Red Hat has two full time KDE people. Than Ngo and Lukas Tinkl. Lukas even moved from Novell to Red Hat.

    http://fedoraproject.org/wiki/SIGs/KDE

    http://fedoraproject.org/wiki/LukasTinkl

37. indigo196 says:[8 April 2008 0:49 AM](#d39af198f658eb0eea33befa20820b2f26421992)

    I noticed that a great many 'good' things attributed to 'Ubuntu' here were really no more than 'good' Debian things.

38. ateimporta says:[8 April 2008 0:51 AM](#088abb3ccd1b1590403ec5780d2dad4fcb49be9a)

    I'm sorry but I join the club that cannot try Fedora because of RPM. I was an all RedHat guy many years back, but when I discovered Debian the difference was huge.

39. apokryphos says:[8 April 2008 1:30 AM](#9ac8598978a6fbb3a6c3be7e530644e90961b1d1)

    Jef: well, it's obviously not just a guess if I gave reasons for them and it's clear I have numbers in mind. Anyway, I decided to make a quick list using http://openSUSE.org/Teams and adding X and OOo developers, in all cases concentrating on strictly \_desktop\_ developers (so not including Kernel, Xen, etc. developers), and I have a list of 91 currently employed developers. Note: this will certainly be a large underestimation -- I didn't bother looking for other teams around the place (i.e. ALSA).

40. [dgoodwin](http://dgoodwin.dangerouslyinc.com) says:[8 April 2008 2:16 AM](#d2976075af51f11c782fda435603e11ca8444c3f)

    Enjoyed this post a bunch.

    After maybe 8 or so years with Debian and then Ubuntu I switched to Fedora about a year and a half ago. There's no denying yum is a little slower than apt but it's amazing how little this matters day to day. Unfortunately if you're going to notice it, it'll be one of your first impressions.

    Dependency hell? These have to be dated impressions, I've never seen dependency problems. I gather Debian and possibly Ubuntu still have slightly larger package repositories than Fedora but quite simply anything significant will be available in all and thus install without dependency problems.

    Grown to love Fedora for it's philosophy, the people behind it, and their dedication to delivering the cutting edge in a package that's still remarkably usable and stable.

41. [Cliff Wells](http://pentropy.twisty-industries.com/) says:[8 April 2008 3:00 AM](#cfbd7ed0fd296fad0537e7fb0b38b260fb48751b)

    If yum were the only option for managing packages on Fedora, I'd be using something else. Smart is \*the\* package manager to beat.

    I can't tell you how many supposedly "rpm-helled" machines that would normally take an hour of --force, --no-deps, etc to get yum or apt-rpm to even consider looking at their rpm databases have been fixed with a simple "smart fixall". Smart is faster than yum. Smart can intelligently \*downgrade\* packages to resolve dependency issues. Smart makes apt look inept.

    I just wonder when the Fedora project is gonna wise up.

42. jef says:[8 April 2008 4:45 AM](#69500702ae73adec8b96f98b4435579cf36c1c83)

    apokryphos,

    That count would be wrong since many of the people listed in the Opensuse site are not Novell Employees. It again appears to me you have no real count.

43. gmureddu says:[8 April 2008 6:00 AM](#007407b4ec3080b4da1da9ebdebab4a5bb1d7487)

    For those that say that Fedora is slower than Ubuntu, and only have tried the betas, is no surprise. All the packages in the distro in beta stages, have debugging symbols turned on and that has a significant performance hit. From the kernel to other tools and desktop software are built this way so testers can provide more reliable feedback. Fedora is the only distro I know that does this (I'm not saying it is the only one that does, though).

    Also, for all the detractors of YUM, I can only say it is getting better, try to deal with .debs without apt and only with dpkg, you'll love rpm then ;-) Sure APT is one fine piece of software, I won't deny that, but at the format level, RPMs are much easier to deal with than .debs, at least in my experience.

44. Kripken says:[8 April 2008 6:10 AM](#4a5fc8a7c9463297e356e4839742d1888f5d6711)

    Well, these comparisons between Fedora and Ubuntu are interesting, but this isn't an apples-to-apples thing.

    1\. As mentioned, Red Hat, which funds Fedora, is orders of magnitude larger than Canonical, which funds Ubuntu. So we shouldn't be surprised that Fedora contributes much more upstream. It's a simple matter of size.

    If Canonical were a large, established, and profitable company like Red Hat, and it didn't contribute significantly upstream, this would be cause to boycott it. But that is not the case.

    2\. Regarding Launchpad. Yes, this is not FOSS. However, it's a bit tangential to FOSS, it's a web service. Complaining that it isn't FOSS is like complaining that Google services aren't FOSS. Personally, I'd like all this stuff to be FOSS, but that isn't going to happen, and I don't hold it (too much) against the companies for not doing so.

    Furthermore, regarding Launchpad: this is an attempt at something quite new in the FOSS ecosystem, "one bug tracker to rule them all," connected to all others. If this works, I believe it might catapult FOSS development ahead significantly. Too soon to tell, though. Regardless, this new idea requires a single central source, at this point at least.

    3\. Finally, I'm a little disappointed at the level of animosity in these responses, "flame away" indeed :) Both Fedora and Ubuntu are worthy projects, I like them both. And infighting amongst ourselves isn't doing our overall goal - of furthering FOSS - any good.

45. jef says:[8 April 2008 7:00 AM](#d50e0fe4d26a62a7c3a05c0d975758ad16cef1b0)

    There is pretty much no upstream project initiated by Canonical. So it is not merely a matter of size. Canonical has more developers working on proprietary launchpad than any open source project.

    "regarding Launchpad. Yes, this is not FOSS. However, it's a bit tangential to FOSS, it's a web service. Complaining that it isn't FOSS is like complaining that Google services aren't FOSS."

    No. It isn't. Repeating it doesn't make it true. Google is not the Ubuntu feature tracking/bug tracking system or translation infrastructure. A Canonical controlled proprietary service is being forced upon Ubuntu which is supposed to be a community distribution.

    You don't need one bug tracker to rule them all and certainly not a proprietary service to rule over open source software. What you need is distributed bug tracking systems that can easily talk to each other.

    The whole point of the fight is that Ubuntu is not furthering FOSS anyway.

46. [dweazle](http://krul.nu) says:[8 April 2008 7:32 AM](#b31c70fa2a85a88d8d73ef2d76a45d5c9ae4de92)

    Both Fedora and Ubuntu are outstanding distro's. But I think what puts Ubuntu on a higher ground is the polish they put into the distro. That is what makes Ubuntu popular: It's (almost) hassle-free. That is where Canonical put all the work, and they're good at it too.

    If you try to play a media file and don't have the right codec installed, Ubuntu will install them. No adding 3rd party repositories and searching for the right packages to install.

    If you visit a website that requires certain browser plugins (like Java or Flash), Ubuntu will automatically install them.

    3D acceleration and wireless cards work out of the box on most hardware without any tweaking or commandline-foo.

    If you try to run a command for a program that is not installed, Ubuntu gives help on how to install the program. That is useful for people not comfortable with the CLI.

    And for me the best part: They follow the GNOME release cycle, so that they don't lag behind current releases.

    It....just....works.

    Sure, Fedora has some advantages too. And for a developer, it might be the better distro after all. But for Joe average, polish is important and Ubuntu does a better job than Fedora, period.

    On the other side, I haven't used Fedora for years. Maybe I should take another look and see how much they've progressed over the years :)

47. Luya Tshimbalanga says:[8 April 2008 8:09 AM](#bb32f37b5a885610e8c8bb1222e0de3d4f92b479)

    "Red Hat and Fedora are currently among the worst distros when it comes to supporting KDE. There is no way to have a KDE desktop by itself in Fedora because all the system tools are built on GNOME -- look how the installer sticks out like a sore thumb within KDE. If Red Hat devs have to lift a finger to accommodate KDE, there's only one answer you'll get from them: too bad, we won't do it."

    Ah complaining? Always complaining yet incapable to step up to make a change? Should you ever visited fedoraproject.org, you will realize the community has formed a KDE group who made a tremendous effort of improving KDE (live CD, Eletronic Lab as few example) since the merging of Core and Extras. You know what is ironic? One of worst distros in term of KDE suppport is now the first to include KDE 4.0 by default.

48. [James Henstridge](http://blogs.gnome.org/jamesh/) says:[8 April 2008 8:18 AM](#0d2a064059c0774bb57f1d9e17f1a7e030be1b5b)

    wingo: If you're going to keep the current ampersand handling, could you get it to present a sane error message? It took a while to work out that the ampersand was the thing causing my reply to fail.

    bla: I am not sure why you think Upstart development hasn't been sponsored by Canonical. Evidence seems to contradict this (e.g. Canonical is a copyright holder).

49. apokryphos says:[8 April 2008 8:28 AM](#775c1833d1862836ed7f69c6aab6aaf2147aaaa3)

    @jef: which part of me saying that I was only counting Novell employees did you miss? I know very well which are community contributors, and they were not counted. Please do not assume the worst.

50. jef says:[8 April 2008 8:43 AM](#1e9eeffeef350bd9b42f17759284e20a64d6bb0e)

    apokryphos,

    I got to when you don't present a good comparison. You say you have the numbers. Tell me clearly, how many Red Hat developers work on the desktop. How many does Novell? What's the impact? It not just numbers. The devil is in the details.

51. Francis says:[8 April 2008 9:56 AM](#f83fee182a3fcb4c62183ae915adaa6c739c9430)

    My prediction is that they have a lot more desktop developers, and probably from just the number I quoted. I never said I had "the numbers", but I did say I gave a lower bound for SUSE employees working on the desktop. Now, you can prove my prediction wrong by finding figures (and I presume this would be easier for you if you're involved with Fedora/RH at all) for RH desktop employees that shows that they are significantly larger, but I haven't seen any indication of that, so I currently don't believe you can.

    I know both (i) quoting a lower bound of SUSE Linux desktop employees is not proof (I'm not going to do a comprehensive survey), and (ii) just quoting developer numbers is an oversimplication if the numbers are very similar to each other. Those two granted, it's clear that a number is an \_indication\_.

    I'd also prefer having this discussion in an easier way (if it will continue) -- do you use IRC? If so, please ping me (same name, freenode).

52. apokryphos says:[8 April 2008 9:58 AM](#d9f87c4de5085b87354e6849791dee23c9ead12b)

    Whoops, I'm the author of the last post.

53. James Morris says:[8 April 2008 10:25 AM](#11b1d0baaa6946f525bfbbe09426cbeab57bcb3c)

    (Interesting to see proprietary software being held up as an example of a defining contribution to FOSS).

    In any case, please let us know why you disable SELinux.

    If you have specific issues with it, please file a bugzilla, or contact the developers via the fedora or fedora-selinux lists.

54. timo says:[8 April 2008 10:45 AM](#bfee9c385f826eed91c960125d1126301b20d6cd)

    If you take the technology glasses away, Ubuntu is the only distribution trying to bring the spirit of free, libre software to the masses, by using ubuntu philosophy as a marketing method to educate people about what free software is.

    For the masses, "open source" or "free software" or "free code" is cryptic, Ubuntu is in a unique position emphasizing the benefit of free software to the humanity etc. That is by far the most important aspect Ubuntu is doing, apart from being a great and polished distro.

    Fedora is also great, especially lately it has began to shine in an all new way. They've been approaching Ubuntu in the i18n of stuff, they have the very latest and greatest FLOSS stuff (Java etc.) installed by default, though luckily also Ubuntu has openjdk/icedtea packaged.

    Regarding "the spirit of ub... libre software", I also like the new marketing efforts of Fedora - the infinity etc. is another great, not entirely technical way of trying to actually build a brand out of libre software. So all the best to Fedora, and I really hope Fedora will rise.

55. keitai says:[8 April 2008 12:18 PM](#d77e0cfbdf9ce4fec35940b4d2565361382c267d)

    I'll migrate to fedora the \*second\* they stop using rpm and yum.

    RH developers appear to be too myopic to see why people hate those tools, a slightly more optimized yum will not help.

    But I am truly grateful to the amount of upstream people RH hires. It is clear that if I needed rock-solid servers with support contracts, I would go or RHEL - and I could be certain that I have \*upstream\* available to solve my issues, not just a random packaging dude.

    I see suse is trying to "compete" with redhat with upstream involvment, but they are not quite there (atleast yet). Also novell seems to favour flashy features presented as code drops timed just before next suse release - Xgl style. RH people seem to \*lots\* of the invisible background work that makes Linux these days.

    Certainly putting distributions to compete with upstream involvement is best for us ALL. oracle, do you hear?

56. jef says:[8 April 2008 1:09 PM](#98572461ccbee75afcea0aa0f4c02ee2e2db6f0c)

    "I'll migrate to fedora the \*second\* they stop using rpm and yum.

    RH developers appear to be too myopic to see why people hate those tools, a slightly more optimized yum will not help."

    Why wouldn't it help. RPM clearly supports more functionality like multi-lib and triggers. Yum works also the same way as apt does and apt is available in Fedora. Explain clearly why or it is you who appears myopic.

57. jef says:[8 April 2008 1:14 PM](#4ac4efdf1833bcddce47f25607fdddc40cd0abce)

    "My prediction is that they have a lot more desktop developers,"

    Your prediction? Seriously? You don't predict such stuff. That's like pulling chickens out of a^w something. Either you have valid data or you don't.

    You said earlier

    'well, it's obviously not just a guess if I gave reasons for them and it's clear I have numbers in mind."

    So spit out those numbers. Just providing one side is not enough to make any valid arguments. Provide numbers for both sides and make a comparison. That would make a good argument.

    I am not interested in a IRC discussion since I don't use IRC.

58. apokryphos says:[8 April 2008 3:01 PM](#90d80e4fb8a790dd5c0c61c6c22eab27271f3537)

    \> That's like pulling chickens out of a^w something. Either you have valid data or you don't.

    Why the heck should I not be allowed to predict the number of developers project X and Y have working on Z? While you don't substantiate that (and yes, completely unrelated analogies don't), I'll keep on predicting. You might not like my predictions, but while you also don't provide figures (I have already made an attempt to give you an estimate of SUSE contributors), your prediction is certainly no better.

    \> Just providing one side is not enough to make any valid arguments. Provide numbers for both sides and make a comparison. That would make a good argument.

    Not easy for me to get a direct figure on Fedora or Red Hat contributors since I'm not involved there and don't know of any of the team pages. If it's easy for you, then I'd be very interested in seeing any figure you have. Of course if you point me to a page with them (like the openSUSE page) I can count myself.

59. jef says:[8 April 2008 3:08 PM](#2fb80e97b1c8948f2f07e017738ee5c042668635)

    "Not easy for me to get a direct figure on Fedora or Red Hat contributors since I'm not involved there and don't know of any of the team pages"

    Then you have no data enough to make predictions either way or you are just guessing. That's the simple point for you. Knowing only one side makes you incapable of knowing which side has more people involved.

60. kripken says:[8 April 2008 4:40 PM](#4abbb0c44b33406bc49b820ece69295629ead987)

    Jef:

    "There is pretty much no upstream project initiated by Canonical. So it is not merely a matter of size."

    Canonical initiated the Storm and Bazaar projects. I don't know about Storm, but I use Bazaar (both for collaboration and for personal stuff like syncing backups), and it's very useful for me.

    Aside from official projects, several FOSS projects sprung up from Canonical/Ubuntu, either from Canonical employees or Ubuntu community members. An example of the first is Upstart, an example of the second is Deluge. In both cases you might say, "well, these projects might have sprung up anyhow" - and that's true. But the fact is, we don't know what might have happened. I can say for a fact, from firsthand knowledge, that Deluge came to be due in large part to the highly effective Ubuntu community.

    "You don't need one bug tracker to rule them all and certainly not a proprietary service to rule over open source software. What you need is distributed bug tracking systems that can easily talk to each other."

    I accept that your position is reasonable. I would say that the other position is also reasonable. I'm not sure where I stand myself, both seem to make some sense. Time will tell.

    "The whole point of the fight is that Ubuntu is not furthering FOSS anyway."

    Well, first, there are the FOSS projects mentioned before. Second, Ubuntu has gotten lots of people to use Linux that might not have, otherwise (I know several such people personally myself). I guess you might say that more users isn't good for FOSS necessarily, but I think it is. I've seen some of those people go on to submit patches upstream to GNOME, for example. So I believe expanding the community is good for us all.

61. jef says:[8 April 2008 5:19 PM](#b4460e67a35bc635e4e4f8a00b20fb38b288630d)

    "I accept that your position is reasonable. I would say that the other position is also reasonable."

    Well, I don't consider the position that open source projects need to rely on a private company controlled proprietary web serivce to do good bug tracking as a reasonable position at all. Sorry. This is just remarkably naive thing to say.

    Canonical has more developers working on this proprietary service compared to the any FOSS projects you have been mentioning and employees doing spare things on their own time doesn't really count as a Canonical contribution.

    I bet that you could have easily used mercurial or git for your backup purposes and it would have worked just fine. Storm isn't a significant contribution by any means.Even Ubuntu relies on sysv compatibility and doesn't take advantage of upstart much so the real story is getting washed away in marketing.

    Sure they might submit patches here and there but I am not impressed with the focus on launchpad and I am not impressed with Canonical treatment of Debian developers. Canonical fails to give due credit to the giants whose shoulders Canonical has been standing.

62. [wingo](http://wingolog.org/) says:[8 April 2008 5:47 PM](#daf0fb46661075f45a58a85ca11253fa3960bc48)

    Well that was fun. But I do share [Kripken's](http://wingolog.org/archives/2008/04/07/fedora-is-the-new-ubuntu#4a5fc8a7c9463297e356e4839742d1888f5d6711) surprise at the level of animosity.

    jamesh, I'll see what I can do regarding error messages; thanks for the feedback, it's appreciated.

    daniels: Whoops, my bad about the Anholt misrepresentation. He's even a FreeBSD guy, no? Guess I got a bit carried away :) Will edit.

63. xanadu says:[8 April 2008 5:50 PM](#02eb96f3283b1eaf12582d1348e98b3e33d88469)

    Wow. The amount of FUD in the second paragraph is crazy.

64. xanadu says:[8 April 2008 5:59 PM](#8edfd1de4e392283d2255b3f32aa8e8d3cbbb23d)

    Could you tell me exactly what's this proprietary software ubuntu comes with? When I installed my ubuntu I had to opt-in to download restricted drivers and codecs myself.

65. xanadu says:[8 April 2008 6:02 PM](#a1cc4e530e415ead9bb67cfd7a6acaeadb00823d)

    Oh, please take a look to Mark's answer https://bugs.launchpad.net/launchpad/+bug/50699

66. kripken says:[8 April 2008 6:10 PM](#b7d62cbcce374472bffeefe696b8f3a1bba01b06)

    Jef:

    "employees doing spare things on their own time doesn't really count as a Canonical contribution."

    Depends on what you 'count'. My point, above, was that Canonical's contribution was in creating a great community around Ubuntu. This then led to things like Deluge and Upstart that might not have happened otherwise. Yes, these aren't direct results of Canonical spending or Canonical employees on company time, so you don't 'count' them. But IMHO these contributions to FOSS are quite valuable, and I give Canonical credit for creating the \*environment\* in which they could arise.

67. Francis says:[8 April 2008 6:10 PM](#afc5873a2e84725069c067ac48416eba0e0443b2)

    @jef:

    \> Knowing only one side makes you incapable of knowing which side has more people involved.

    Fortunately I don't \_only\_ know one side then, but only have \_figures\_ for one side. I know that SUSE has more paid developers than Ubuntu, but I have no numbers for Ubuntu too. With Red Hat I \_know\_ it's closer, but I still \_predict\_ that it's higher (for desktop developers).

    In fact, the position is mainly based on the fact that:

    \\* I predict SUSE has a similar amount of GNOME developers to RH (not completely unjustified -- this is by looking at the components link provided)

    \\* that in X, FreeDesktop and other components there isn't a HUGE gap in the number (since I know there are SUSE developers working on core components in many of those cases)

    \\* that SUSE for sure has far, far more OO.o and KDE developers (two of the largest desktop components)

    So: (conclusion) it's very hard to see where the numbers for RH/Fedora would be made up. As there aren't very large teams of developers working in other areas.

    It's a prediction without FULL figures on RH, but it is reasoned and it's unclear that figures is a necessary requirement for a prediction.

    And all this because you wanted a very detailed argument into how I reached a conclusion that you evidently don't think is grossly incorrect :-)

68. John Mahowald says:[8 April 2008 6:47 PM](#e29f54170ce4e4d6c20372faeab1b57f90b8b364)

    "Could you tell me exactly what's this proprietary software ubuntu comes with? When I installed my ubuntu I had to opt-in to download restricted drivers and codecs myself."

    That this comes from a restricted Ubuntu repository and the non free enabling tools shipped with Ubuntu is the distinction. Fedora leaves that entirely to 3rd parties.

    I'd say both Fedora and Ubuntu camps can agree that free is better than proprietary. Fedora just won't touch restricted stuff at all.

    Which distro is easier to get the proprietary stuff for users who do want it, and how this might sway their distro choice I cannot say.

69. jef says:[8 April 2008 8:40 PM](#cf6c12fb085f425587b4db56ff3e7b9ec08a2d59)

    "It's a prediction without FULL figures on RH, but it is reasoned and it's unclear that figures is a necessary requirement for a prediction."

    You have presented zero figures on the RH side.

    "And all this because you wanted a very detailed argument into how I reached a conclusion that you evidently don't think is grossly incorrect :-)"

    I think you have only half baken data on one side and grasping for straws and sticklers. Good luck.

70. Jonas says:[8 April 2008 10:25 PM](#2a21708d99815f952e1b4e220aef4093be26ed67)

    Well, I just have to put in my two cents in this dicussion.

    First off, I think that neither Fedora or Ubuntu gets it all right (or rather, the people advocating one or the other). They both have their pluses as

    well as minuses. Neither is perfect for everyone or we wouldn't have this discussion in the first place.

    That being said...I think it's rather myopic to switch distro if the major reason for doing so is because the distro switched from makes it too easy to

    activate the proprietary stuff. Why? Firstly, while it's easy to activate say w32codecs, Adobe's flashplugin, or the nvidia driver Ubuntu doesn't require

    you to do so. And just because Fedora or some other distro doesn't make it as easy doesn't mean it's impossible. If you want it, you can have it.

    Secondly, fewer and fewer Linux-users gives a damn whether they should use ogg-files or mp3-files for example. All they want is a computer they can use

    for whatever they see fit, and if that involves using non-free stuff...well, guess what? They will use whatever gives them that freedom - and a foss-only

    computer won't provide that to them.

    Which brings me to my next point. Ubuntu is popular for a reason. It's relatively easy to get the proprietary stuff if you want or need it, and let's face it:

    some people do need it. Not just want it for some personal reason (important though as that is) but actually NEED it to be able to use their computer for

    productive uses. Try as might, I just can't find a reason why it's bad to make Linux more palatable to non-FOSS-geeks.

    I've used Linux for years, and I still find the insistance that every Linux user should subscribe to the FSF-motto contra-productive at best. If FSF-approved

    only software meets your needs, great. Good for you but don't try to make it sound like it's good enough for everyone until the nv driver is as good as

    the nvidia one, or until gnash can negate the need for Adobe's flashplayer, or until there's a FSF-equivalent of the w32codecs or the AdobeAIR runtime as just a few examples.

    Just face the facts: people in general won't switch to Linux if there's a perceived drawback to it. And insisting by imposing artificial barriers such as

    "Ubuntu is crap because it insists that everyone should be able to use Linux whether they agree with FSF or not" is doing just that: limiting Linux

    to those who already knows of Linux and not potential users and/or contributors. And honestly, the more users we have the better off we are regardless of what

    distro those users are using.

71. jef says:[8 April 2008 10:33 PM](#14ba0f5dfdaadb919b9a9941c4a8a8200cac1e76)

    "I give Canonical credit for creating the \*environment\* in which they could arise."

    I don't know about you but If my employer gets all the credit for work I do on my personal time, I would be totally pissed off.

    "Which brings me to my next point. Ubuntu is popular for a reason. It's relatively easy to get the proprietary stuff if you want or need it,"

    Windows is way more popular and makes it much more easier to get proprietary stuff. Popularity does not say anything about whether it is right or wrong or good or bad.

    If Ubuntu facilitates violation of the GPL license of the Linux kernel via proprietary modules, I will hold them accountable for the damage they are doing either legally or morally.

    http://www.kroah.com/log/images/ols\_2006\_keynote\_12.jpg

72. Jonas says:[8 April 2008 10:56 PM](#3f6c1eedfd6d9f8fff3d1a33b00f193cad3e97b9)

    "Windows is way more popular and makes it much more easier to get proprietary stuff. Popularity does not say anything about whether it is right or wrong or good or bad."

    No, it doesn't. That's true. But it does say something about WHY some people (myself included) prefer Ubuntu over the competition. It's easier to get the functionality and speed you require. Would I be able to get a similar result out of say Fedora, Suse, or LFS? Sure, but with more work. And as the cliche goes: time is money.

    And that image is quite possibly the stupidest thing I've ever seen, and a good reason why Linux as a whole isn't taken more seriously as a contender in the desktop arena.

    If one take that stance to the extreme: OpenOffice is illegal because it facilitates using closed-source formats.

73. jef says:[9 April 2008 0:34 AM](#132c64876d7b277916fd3b7011e091c9d47f6d64)

    "But it does say something about WHY some people (myself included) prefer Ubuntu over the competition"

    The same thing it says about why a whole lot more people who prefer Windows?

    "And that image is quite possibly the stupidest thing I've ever seen, and a good reason why Linux as a whole isn't taken more seriously as a contender in the desktop arena."

    The image you called stupid comes from a keynote presentation by a prominent Linux kernel developer. You can't ignore the license to win a market. Neither should you as a user. That is what Ubuntu users don't seem to understand. Respect for the foundation of the ideas that makes Linux what it is.

    "If one take that stance to the extreme: OpenOffice is illegal because it facilitates using closed-source formats."

    There is just no comparison. GPL programs can interact just file with formats that are reverse engineered. GPL programs cannot have proprietary add-ons however. Read the license before spouting nonsense.

74. [James Henstridge](http://blogs.gnome.org/jamesh/) says:[9 April 2008 1:07 AM](#9b0cfe342a8f6257df92520085a4a12a55c93de1)

    jef: Canonical does not get ownership of code I write off the clock. I assume the same is true for Scott.

    Canonical is a copyright holder for Upstart because work was done on it on company time. It was written to fulfil the needs of Ubuntu for a better init, after all.

75. Jesse says:[9 April 2008 1:21 AM](#ac126c99cc38e3311efa76bf83ec12ee2425a5ab)

    I was a loyal Debian user for years and then was really happy to switch to Ubuntu when it came out, because I prefer the Gnome desktop as well (and I was always running unstable to get recent version of software). I remember installing FC5 at first because I had tried to upgrade my system to the newest version of Ubuntu at that time, and there was a problem with the installer media I believe. I could have just downloaded another copy of Ubuntu and tried again, but decided to try Fedora...and I haven't looked back since.

    I like Fedora because it's cutting edge, and the developers are serious about open source in a way I appreciate. I freely admin to running a couple closed source end user apps (which run fine) on my system, and I don't think there's anything wrong with that. As a policy, however, it doesn't make sense to include proprietary shit in a complicated system that's built in a distributed, open source fashion. It makes debugging harder, and can create ugly, wicked, evil problems. Fedora's policy on this (not to mention the amazing developers employed by Red Hat) gives me more confidence in their system, and my experience has generally confirmed that feeling. It feels really solid, and if you look at the advancements they've been making in recent releases, it's quite impressive and exciting.

    I don't begrudge Ubuntu at all for their concerns and focus. I think its great that they are attracting new linux users. They're doing a good job at it too. I think that's a great thing for the linux community and open source software as a whole.

    PS, for everyone who keeps saying that if only Fedora would only switch to dpkg and apt, GET SERIOUS, folks. Yum is now quite good, even if it's slower than apt. There is practically no difference between typing 'apt-get install foo' and 'yum install foo'. In both cases, all dependencies are resolved automatically! And its a bloody package management tool! Do you sit and install/uninstall packages all day, marveling at apt's speed?

76. Kripken says:[9 April 2008 5:17 AM](#eb16a199bc72f2fcf37f42ac16834b4055feedfc)

    Jef:

    "I don't know about you but If my employer gets all the credit for work I do on my personal time, I would be totally pissed off."

    Well, I am one of the two founders of the Deluge project, and I'm not even a Canonical employee, just an Ubuntu community member (at least at the time I was). As you saw before, I freely credit Ubuntu and Canonical for creating an environment in which my project, Deluge, could occur.

    Certainly they do not deserve 'all' the credit, and in fact they have asked for none of it. But without them the project would not have existed, and I am grateful to them for that.

77. jef says:[9 April 2008 1:00 PM](#502ca4391e2ce54855a634e1879b3e3cf78d4685)

    "jef: Canonical does not get ownership of code I write off the clock. I assume the same is true for Scott."

    They and only they own the code you write on the clock for Launchpad. I don't how much of your time that is but all that time is deprived from working on the community for upstream code. Not very community friendly I am afraid. The fact that you are willing to stand up for it instead of opposing it is a loud message.

78. apokryphos says:[9 April 2008 1:42 PM](#c76deb03febfa51e669992771451f63edc494f4b)

    @jef:

    \> You have presented zero figures on the RH side.

    Perhaps, but I did suggest there were easy comparisons in many of the projects. Since I know the SUSE numbers in this case, I can also know something about the RH numbers. If you think that Red Hat's contribution in the major projects (KDE, X, GNOME, OO.o, FDO) does not give an \_indication\_ of their overall desktop developer numbers then we can disagree and I won't argue with you. I don't think you'll find anyone to agree with you though.

    \> I think you have only half baken data on one side and grasping for straws and sticklers. Good luck.

    And I think you haven't said that you disagree with my original statement too, so no problems there :-)

79. jef says:[9 April 2008 5:22 PM](#fca0d3f6a1aeae8b11072943143ed2fe94109a8e)

    "Perhaps, but I did suggest there were easy comparisons in many of the projects"

    If it is so easy, why haven't presented any comparisons? Is it because you don't have a clue?

    "And I think you haven't said that you disagree with my original statement too, so no problems there :-)"

    Your original statements are not backed up with anything substantial. When you bother to make a good comparison, then I will evaluate and let you know whether I agree with it or not.

80. [Valent](http://kernelreloaded.blog385.com/) says:[11 April 2008 1:44 PM](#ce1463818bdb5543d500fd3b1546a5e24de56c13)

    Great article!

    I have seen how apt-get and yum work on my machine and to me it seams that apt-get is not much faster than yum, maybe slightly - but I see a major difference in speed when dep and rpm packages are installed - and dep leaves rpm in dust!

    It is easily twice faster to update and install packages on Debina based distros than on Red Hat based ones. I believe that is not because yum is slow but because rpm packages are slow to install and upgrade compared to deb ones.

    Valent.

81. [Hannibal99](http://hannibalsite.blogspot.com) says:[11 April 2008 6:31 PM](#b99ec52fe9413309081a24e5c9bbcaf1c227b336)

    I had moved from Ubuntu 7.10 to Fedora 8 - and its quite good, and I believe it will be better as far we will go. Anyway I have only a Pentium D 3.4 and Celeron 1.86, but it is not slower as Vista, cos I have dualboot. I am waiting for Sulphure too.

82. [Michael DeHaan](http://michaeldehaan.net) says:[13 April 2008 1:18 AM](#a83d742c86ace3403b0d2b5d2feeb42b47c98515)

    It would be outstanding if Ubuntu /did/ take a greater role in sponsoring upstream developers and contributing to the free software community.

    I would definitely encourage them to start by opening up the code behind all of their infrastructure -- i.e. Launchpad, as well as submitting more patches to things they find interesting. Currently it seems that most Ubuntu employees are merely re-packagers -- which is acceptable for a small company, but not so for the amount of press they like to generate about themselves.

    Linux distros are /basically/ the same, but what differs is how their communities are lead and run and how open they are. I would love to see Ubuntu open itself up more and give back to upstream at a greater rate.

83. not\_important says:[13 April 2008 2:34 AM](#3e9c87bc5257cc38adb3d1568e7b63e3b57c5f91)

    I left Fedora for Ubuntu. I was particularly annoyed by two things -

    1) Forcing personal beliefs (100% OSS, 4K Stack) on end users who want to get things done and not feel pissed in the process - 4K Stacks bit me very badly when using 100% OSS, in kernel.org file system drivers resulting in very nasty crashes.

    2) I reported dozens of bugs - no one \_ever\_ cared except for automated new release responses that asked to verify if the bug still existed in new release

    Additionally yum sucks is the reality - it is slow, lots of times the mirrors are dead slow and result in timeouts - this sort of suckage never happens with Ubuntu. I am not talking about codecs.

    I have never had any kernel related issues with Ubuntu kernel, bug reports are usually well attended and I am not forced to do things the few distribution idiots feel are the one real way - I am not offered proprietary drivers by default but if there is no OSS driver available hardware drivers application lets me enable a prop driver in one click - no fuss. Same for codecs.

    I can use CentOS if I had to use a RPM based RHEL compatible distro - works fairly well but I cannot stand Fedora any more.

84. jef says:[19 April 2008 2:51 AM](#00d5a3d64856e0ed0ab8ac120964929a5dfa1b24)

    "Forcing personal beliefs (100% OSS, 4K Stack)"

    BS. This change is in the upstream linux kernel. Nothing to do with personal beliefs.

    "I reported dozens of bugs - no one \_ever\_ cared except for automated new release responses that asked to verify if the bug still existed in new release"

    What are the bug reports?

    "Additionally yum sucks is the reality - it is slow"

    Which Fedora release? Sounds like you never tried anything after Fedora Core 5 or something

85. [James Cassell](http://www.jamescassell.com) says:[20 April 2008 3:34 AM](#f34c050bfe3a63e3ef2698d7c547d31ab40eb5df)

    @not\_important: you should try yum-fastestmirror plugin -- it chooses the fastest mirror, and makes things move along much more quickly.

86. [Pablo](http://Alufis35.uv.es/iranzo/) says:[20 April 2008 6:24 PM](#f818aa5094e833e3b1f490f275d0b5d39678295c)

    It's true that Ubuntu deserves credit for getting users to Linux, nobody could disagree on that.

    It's true too that Linux and Free and Open Source Software is now what it is because of it strong ideas about "Freedom", "GPL", etc

    For the end user could be easier to stick with privative software to get the things done, on my side, I prefer to see which hardware manufacturers create or provide information for creating open source drivers, even in the case of having to pay more for it.

    Rewarding companies that go opensource is good, many of them create the opensource pieces of software that stick together and make the idea of FOSS grow.

    Easing users to use propietary drivers, codecs, etc, makes things faster for them, but this is somewhat, stopping people to realice what open sources is, what is it's origin and what's the problem: Manufactures not caring about their operating system choice, they just release a binary driver and the users will manage to make it work. This is what removes credit from the ones that really collaborate.

    And regarding users number... well, as you said, FOSS is about community, collaboration, etc. I want that improvements made on any piece of software could benefit everyone else: for learning, for it's use, whatever, and as contributor, I could help with coding, with translation, testing and bug reporting, documentation, irc for solving doubts....

    ¿is it true that the actual grow of linux users base (all distros/flavours) has the same rate of growth as the real users doing the work (paid or not)?

    If Linux grows with a big user base which doesn't contribute to the idea of FOSS, but just "using it", this will stop the Linux success, because the lack of "principles" that one day made it be what today is.

    Regards

87. not\_important says:[27 April 2008 9:44 PM](#e052b6a6795214190880f0b89ad94605eedf20bd)

    @jef

    "BS. This change is in the upstream linux kernel. Nothing to do with personal beliefs"

    I don't care if it is upstream or down stream - if it bites me and not the "enterprise" customers I would be pissed. In short they should not make distro users a testbed for enterprises - recognizing that Fedora users may have different needs than the need to test RHEL would be good.

    Oh and the 4K stack change recently was pushed into upstream with no discussion and many people disagree with the change (Prev x86 maintainer , -mm maintainer , SGI XFS folks etc.) which is very bad because it was in the same spirit of Fedora - all we care is our enterprise customers and we will use mainline to test 4K stack even if end users suffer. \_This\_ is BS.

88. Anon says:[19 May 2008 8:08 PM](#77bac3839c513037e80bbcc9b14a21c684d7c241)

    I'm not even gonna read this page cos it's literally the narrowest page ever. I can barely read on 1600x1200

89. Vladimir says:[24 May 2008 9:06 PM](#9f96611dcabfb3d5e6ebfc5a5580219e9263ade4)

    Ubuntu (Gobuntu - derived from Ubuntu, uses only open-source non-restricted software)\* and Fedora are both great distributions.

    It is pointless to argue which is better, because they both have they pros and contras.

    And they are both truly open-source...

90. Joe Whyte says:[24 October 2008 9:26 AM](#b2bf0dc5e970597a172807946babc741b793c33e)

    My friends have ben trying linux for years but i wasn't tempted until i stumbled upon ubuntu hardy. enuf said.

    However i can't understand the hatred that exists between distros. My understanding, correct or not, is that each distros has done its bit to advanced the functionality and poopularity of the product(linux) and each deserves credit.

    whyy do you lot hate ubuntu so much? personally,while i applauded linux all these years, i just wouldn't touch it until ubuntu came along and then i could finally get rid off worthless winblows.I think the competitive streak is so strong that since we blew winblows out of the water and they are no longer worthy of our disgust we have turned this hatred on our own selves.

91. [Joshua](http://joshuajava.wordpress.com) says:[27 October 2008 11:54 PM](#5cf0e2f18c6e248a39b615a94845c0d369540cac)

    Wow cool, I didn't know so much lately about the progress between Ubuntu and Fedora and I want to install Linux on my laptop now and confused which one to use. Thanks for this info. :-)

92. gnuman says:[12 February 2009 2:37 PM](#fe711175c43f9466d927eee06adc17875f8b9dc2)

    Josh, save yourself the stress - go ubuntu. It's "install and use"..not plug and play around for a month before use.

93. Event\_Horizon says:[14 March 2009 4:34 PM](#a73935bbedc8df2ee5af36eb3bef24bdb1cd0d96)

    FREE free free - give it to me for free! Who gives a rats ass about those that actually contribute! Isn't it just grand how everyone bitches that something isn't right or should be faster, and why isn't it finished yet?!?!

    Quite frankly I don't know how the coders put up with all this bullshit without slapping a few of you upside the head and reminding you ITS FREE you little pisshead!!

94. [wingo](http://wingolog.org/) says:[14 March 2009 6:01 PM](#213ee554f4232f15aee1a09080172f3ff2a0fce2)

    Next stop, Godwin's Law!

95. 007X says:[12 June 2009 6:21 PM](#42e6c2f0c726823c63b20f9f2ef623772997956b)

    It would be a good thing if there were less distros, and common repositories.

    Fedora, Ubuntu, who cares as long as it works and it offers easy migration from Microsoft.

    For people who have suckled from the Microsoft teat since the beginning, an alternative that is viable and works would be good. MacOSX rapes people as much as Microsoft, and with Linux users fighting amongst themselves, well Microsoft continues to rule.

    What would happen if people really supported the Linux Distros developers, and yes with a little money (everyone has to live!! Would you work for nothing?) With a huge community backing, you could have a fairly priced alternative, providing a real answer to the problem of Microsoft.

    But everyone wants something for nothing, then complains when it does not do exactly what you want it to do.

    I started using Ubuntu, and have had issues, Windows pisses all over it, but I have managed to get it to do what I need it to do. So I say thank you for giving me the opportunity to leave Microsoft behind.

    Stop complaining!

96. Mr. Swillis says:[2 December 2009 6:55 AM](#e4c1962f453c3ce92d41c6893751340d05305c49)

    Ok, all I have to say is... please, for the love of god, don't ever use the term "RPM Hell" in a discussion taking place in modern times. I'm sorry, but the term you refer to simply doesn't exist anymore. RPM Hell was a term used to describe a common scenario that existed \*before\* online-repo-base-package-managers existed. Back when an RPM was something you downloaded from the internet like an EXE and attempted to install manually.

    That was yesterday's world (like pre 2000), so you don't get to use that term anymore. I'm sorry, you're just to new to be allowed to use it. It's extict. Doesn't exist anymore. The only time "RPM Hell" manifests itself in current times is when a user (god only knows for what reason) decides to download and install an RPM manually instead of using YUM or some other tool. And guess what! The same issues will come up with DEB packages if you do that. So please... for the love of god... don't ever use the term "RPM Hell" unless it's actually in reference to a \*historical\* time. It is NOT an acceptable term for modern times.

    Swill

Comments are closed.
