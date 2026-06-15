---
title: towards a gnu autonomous cloud — wingolog
url: https://wingolog.org/archives/2010/04/10/towards-a-gnu-autonomous-cloud
published: "2010-04-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/04/10/towards-a-gnu-autonomous-cloud
---

# towards a gnu autonomous cloud — wingolog

## [towards a gnu autonomous cloud](/archives/2010/04/10/towards-a-gnu-autonomous-cloud)

10 April 2010 10:54 AM

- [gnu](/tags/gnu)
- [autonomy](/tags/autonomy)
- [anarchy](/tags/anarchy)
- [distributed](/tags/distributed)
- [cloud](/tags/cloud)
- [privacy](/tags/privacy)

My previous installments on November's GNU Hackers Meeting ( [hither](//wingolog.org/archives/2009/12/13/gnu-gnome-and-the-fsf), and [thither](//wingolog.org/archives/2010/04/02/recent-developments-in-guile)) touched some topics that were important to me, but not as important as the one I'll mention tonight.

Tonight I want to talk about autonomy and the internet. I'll approach it from a roundabout direction.

**the facebook problem**

Many of you probably know someone who has had their Facebook account disabled. [Here](http://blogs.gnome.org/rodrigo/2010/03/26/facebook-account-disabled/) are a [couple](http://www.jonobacon.org/2010/03/26/facebook-account-disabled/), and [here](http://www.google.com/search?q=%22facebook%20account%20disabled%22) are some thousands more. While I'm probably not the best person to speak of this, as I don't have a Facebook account, it's quite irritating to have this happen. It's like you've been unpersoned.

Beyond the individual indignation though, what is really important (and sometimes missing) is a more universal indignation: never mind *me*, what gives a corporation the right to unperson *anyone*?

Sure, I hear you arguing that it's their services, bla bla, but the end of it is that when you use Facebook, you lose autonomy -- communication and identity are needs just like any other.

I might be going out on a limb here, but consider [Article 19](http://www.un.org/en/documents/udhr/index.shtml#a19) of the fine UN Declaration of Human Rights:

> Everyone has the right to freedom of opinion *and expression*; this right includes freedom to hold opinions without interference and to seek, *receive and impart information and ideas through any media and regardless of frontiers*.

Emphasis mine, of course. What I'm saying is that you shouldn't depend on the government or a [corporation](http://ascii.textfiles.com/archives/1717) or any other entity outside your actual community to be able to actualize these natural rights.

**and further: [article 12](http://www.un.org/en/documents/udhr/index.shtml#a12)**

Though I don't like the wording of this one as much as the previous article, nor the [gendered pronouns](http://aetherlumina.com/gnp/), check it:

> No one shall be subjected to arbitrary interference with his privacy, family, home or correspondence, nor to attacks upon his honour and reputation. Everyone has the right to the protection of the law against such interference or attacks.

As an American living in Europe, it has taken me some time to appreciate the European focus on privacy. I don't think people in the States understand the issues as well as people do here. OK, so your parents/grandparents lived through fascism: so what?

On a personal level, whether you're an [industry insider](http://www.businessinsider.com/warning-google-buzz-has-a-huge-privacy-flaw-2010-2), someone [avoiding an abusive relationship](http://fugitivus.wordpress.com/2010/02/11/fuck-you-google/), or an [Earth Liberation Front](http://en.wikipedia.org/wiki/Earth_Liberation_Front) activist, privacy is terribly important. It's not an exaggeration to say that to cede control over your privacy is to cede control over your identity.

But organizations that control your data on this level usually aren't stupid enough to let you know, or make you think about it. All that is left is a dull throb of [database scandals](http://en.wikipedia.org/wiki/AOL_search_data_scandal) and [terms-of-service changes](http://consumerist.com/2009/02/facebooks-new-terms-of-service-we-can-do-anything-we-want-with-your-content-forever.html) and [wiretaps](http://en.wikipedia.org/wiki/NSA_warrantless_surveillance_controversy).

The problem is not the existence of malicious people: the problem is that the your data is out there. All it takes is one nosy person, or one controlling governmental agency (cf. [A](http://en.wikipedia.org/wiki/Ministry_of_Public_Security_of_the_People%27s_Republic_of_China), [B](http://en.wikipedia.org/wiki/National_Security_Agency#ECHELON)), or one corporation wanting to monetize (cf. all of them).

There are simply no safeguards. There is nothing you can do if you want to be a part of the modern web to protect your privacy. Your data on servers is always available to wiretap, and subpoena if necessary. Your data is not your own.

**origins**

Both of these problems (unpersonage and privacy violations) stem from the fact that you rely on someone else's computer to fulfill your personal needs. RMS wrote about this in an article entitled [Who does that server really serve?](http://www.gnu.org/philosophy/who-does-that-server-really-serve.html), and I agree with all of his points.

Stormy Peters' recent article, [10 free apps I wish were open source](http://stormyscorner.com/2010/04/10-free-apps-i-wish-were-open-sourc.html), illustrates many of these misunderstandings. Besides the misleading terminology, what if Gmail were AGPL-free? Would that protect users against the recent Buzz fiasco? No, because users are not in control of the software they use. Users should be able to modify the software they run; if they cannot, due to that software running on another machine, they should not run software on another machine.

I don't think Richard's article goes far enough. As I mentioned above, the problem is that your data is just "out there". Let's postulate an AGPL Gmail that also allows me to run my own Gmail software on Google's network. While this would meet the Free Software definition, it still harms me as a user, because anyone who has access to that server has access to my data.

Besides that, there is the practical difficulty, in that Facebook or Google would never allow you access to the programs that run on your data in that way.

What I'm building up to is the idea that *the client-server paradigm is fundamentally incompatible with autonomy*. Growing your own food is better than sharecropping, better than "web 2.0".

**what shall we do, sir wingo**

A fundamental problem requires a radical (adj.: *to the root*) solution. As is often the case, the seed of a solution has been with us for a long time: [public-key cryptography](http://en.wikipedia.org/wiki/Public-key_cryptography).

Geeks have long enjoyed mailing each other [signed and/or encrypted](http://www.gnupg.org/) mails, allowing private communication over insecure networks, relying on webs of trust to ensure the identity of the sender. Asymmetric cryptography allows you to send and receive private messages over insecure channels, like the internet.

I won't belabor the point, as most of my readers have seen GPG; it is [the Right Thing](http://www.jwz.org/doc/worse-is-better.html). But what would GPG-style interactions mean in the context of Facebook?

All of you are probably cringing at this point, imagining the complexity and security implications. But let's bask in that moment for a while, shall we: if it were the case that fellow facebooklicans sent you private messages via GPG, being able to view them sensibly over the web would imply that the Facebook server would have your private key.

Extrapolating this farther, the very set of your "friends" is a kind of private data. If this data were properly encrypted and signed against your private key, to present the standard facebook view that most people know would again require your private key.

In the end, [you can't have web services that access private data](http://www.daemonology.net/blog/2010-03-11-zumodrive-rolls-a-hard-six.html). Not if you want privacy, anyway.

**an autonomous facebook?**

To preserve the privacy of your identity, you should never send your private key over the wire. This is well-known. But if you are to do computation on your social network, as facebook.com does, then it follows that such computation must be done local to the user's machine.

But all of facebook on your local machine? Surely you're joking, Mr. Wingo! Well, yes and no. Obviously the answer is not "let's everyone download a program from facebook and run it locally with your private key as an argument". Not quite, anyway.

One good part of the so-called "web 2.0" is that I can code foo-anarchist-commune.org's web site in Scheme and no one is any the wiser. It's easy to deploy in today's environment; deploying e.g. a new facebook experience should not cause me to have to click something to install a new binary.

So, the constraints are:

1. My key pair is my identity. My public key may be distributed, but my private key must be private.

2. Since computation needs my private key, computation must happen locally. Viewing an "autonomous facebook" implies running a program on my local machine, with access to my private key.

3. Since an "autonomous facebook" would be useless without other people, I need access to other people's information, I need a network too.

We can already draw a picture of what this looks like. Let's assume that the end-user experience is still via the web browser.

[![](//wingolog.org/pub/autonomous-cloud.png)](//wingolog.org/pub/autonomous-cloud.svg)

I think that my paranoid readers know where I'm going with this. My technically-minded readers will be flabbergast, perhaps, at the enormity of the problem of implementing facebook under such constraints. How does my facebook know that it's participating in a network? How does it know about my friend Leif? How does it get updates? Where is the database?

**autonomous data model**

Well, one thing is clear: someone needs to hold all of that data. Who to do it? In the case of my data (my photos, my messages to others, etc), I should be the one, as it makes me more autonomous. Everyone needs to seed their own data on the network.

I might choose to seed my data from multiple locations, for reliability. Beyond that, nodes might cache information that is routed through them.

One way to implement such a distributed store would be git-like, with content-based addressing and consistent hashing; or like bittorrent. It would have efficiency advantages. I thought for a while that this would be the solution, but GHM folk brought up the privacy argument, that your pattern of network access is too revealing.

So my current thought is to use [GNUnet](http://gnunet.org/) somehow. I'm not sure how this will go, but it's worth a try.

**new operating system**

Currently, to deploy a web application, you have to pay for servers and bandwidth, and this eventually causes your interests to diverge farther from that of your users. With an autonomous cloud, you could instead deploy web applications using the compute power and bandwidth of leaf nodes -- the power of the people using your software.

This would drastically lower the hacktivation energy for a new project. The little green sandbox above starts to approach a new kind of operating system, even -- a new program to run your programs.

Obviously, I'm thinking Guile would be a fine runtime for the sandbox, to run programs written for Guile -- in Ecmascript or Lua or Scheme or Elisp or whatever other languages people implement for Guile. The user would receive the source code, and running it would automatically compile it on their machine. The application source would also be available to modify and redistribute.

Having a sandbox for mobile code also raises the possibility of interesting mapreduce-type operations, to index the distributed data store.

Firefox could be modified with a plugin to add a new addressing mode, which would go through the HTTP server running locally to your machine. You would be browsing the "autonomous web".

Of course, since the whole thing is based on protocols, one might substitute the Guile environment for something else; or write an alternate interface to Facebook that works over a console or presents you with a native (e.g., Clutter) interface.

**related work**

There have been loads of people thinking these ideas; none of them is new.

My ongoing use of the term "autonomous" is a nod to anarchists, and to the [autonomo.us](http://autonomo.us/) group. Autonomo.us would be a great organizing place for work around this, but their list server is a bit moribund; somewhat ironic. Perhaps we can return life to that group, though.

[GNU Social](http://groups.fsf.org/wiki/Group:GNU_Social) is a project to make a free-as-in-freedom social network. I think it's a great initiative, and it's probably the place to go if you want to build an alternative to Facebook right now.

GNU Social has made the decision to just get something working. This is the right thing to do, IMO; but near-term solutions should not prevent concurrent research for the long-term. In the end if making an autonomous cloud turns out to be possible, perhaps we can rebase GNU Social on top of the autonomous infrastructure.

Is there something else I should really be looking at? Let me know! I don't think one can ever do a full survey of this field -- better to just start hacking -- but I'm interested in good ideas, especially to the data storage and access problem.

**plan**

All of this is a bit pie-in-the-sky, but I am going to see if I can work up a proof-of-concept for the upcoming GHM in July. If you are interested in helping this project, probably the best thing to do is to code up some little demo application using GNUnet or some other store. Once you have that, drop by to see me in #guile and we'll talk.

Comments welcome!

## related articles

- [storage primitives for the distributed web](/archives/2011/03/24/storage-primitives-for-the-distributed-web)
- [fscons 2011: free software, free society](/archives/2011/11/28/fscons-2011-free-software-free-society)
- [bart and lisa, hacker edition](/archives/2011/03/19/bart-and-lisa-hacker-edition)
- [slouching towards bethlehem](/archives/2010/04/11/slouching-towards-bethlehem)
- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

### 25 responses

1. [Zash](http://zash.se/) says:[10 April 2010 11:21 AM](#7d248fafef8c79572984e95d4d289d160c77a7ae)

   Have you looked at onesocialweb?

2. [wingo](http://wingolog.org/) says:[10 April 2010 11:34 AM](#87010e2e26f4a5e1c9f721cec011b88df153c1f8)

   Zash: Nope, hadn't seen it. Thanks for the pointer! (Though: Vodafone???)

3. Colin says:[10 April 2010 3:26 PM](#ad0f4538955b748511ff0730338e9480fe9360af)

   DSNP is another option: http://www.complang.org/dsnp, although it's more of a trust network for information on privately owned servers than a P2P idea.

4. [Steve](http://www.mynext.co.uk) says:[10 April 2010 5:12 PM](#85a493e0ea89f627b0188cb69d4b51b1dad42b80)

   An open facebook would be lovely and one where you can control your privacy properly would be brilliant. Afraid neither are coming soon =\[

   Think the hard thing will be breaking this monopoly as it was one of the first websites to get people on mass to use their real names rather then an alias.

   Recon the point that will break the facebook monopoly through will be a massive harvesting of real name and dob, amazing what you can do with that

5. mdgeorge says:[10 April 2010 5:50 PM](#3bf60310bce85a21665bbe9b76851ec5c29100b0)

   A shameless plug for a project we published in SOSP that was designed with these goals in mind:

   http://www.sigops.org/sosp/sosp09/papers/liu-sosp09.pdf

6. [Ivan](http://ivan.unixdaemons.com/blog/) says:[10 April 2010 6:03 PM](#9dc7a7828bf5fb86b36648b5cb2c5f5f3093cb7b)

   Very interesting ideas.

   Privacy is going to be tough in the face of an eavesdropper that does network monitoring. We might want to keep continuous bogus traffic flowing between nodes to hide the actual communication in noise...

   A very important recent discovery in public key crypto is the concept of "partial disclosure" of information contained in a certificate.

   This would be akin to a "privacy policy" for my data.

   Here is a book on that:

   http://www.credentica.com/the\_mit\_pressbook.html

   Brand's company got bought by M$ a couple of years back, but we can probably re-implement most of his ideas from scratch...

   As for a simple and practical first step towards a distributed facebook check out the following proposal:

   Define: My set of friends F = {f\_1, f\_2, ..., f\_n}.

   For each of them, I have on my own server a copy of their pk (publc key).

   Now I decide to post a new status message m or a picture or whatever people do on facebook.

   Instead of posting a single message, I acturally create n different copies of the message -- each one destined for one frind in F, and encrypted with his pk.

   My friend's personal facebook server's poll (or get notified by?) all other servers for new messages destined to their user. Peer nodes can act as relays since the messages they forward are unreadable except for the actual intended receiver (the owner of the secret key sk).

   The efficiency is terrible, but for text the massive duplication won't hurt so much...

   As for a philosophical question... do you think renting a VM from a cloud provider is NON-FREE? What if I install my own software and have complete control from OS level up?

   How much do I need to trust the cloud provider?

7. forkandwait says:[10 April 2010 6:11 PM](#f9f5fdd1f7f277436a6355e0e7afe76c0978b5ed)

   http://news.ycombinator.com/item?id=1255058

8. [Patcito](http://shapado.com) says:[10 April 2010 8:13 PM](#0ac2b27d85131df23633da965114eebedc8e0a41)

   Doesn't ostatus already solve the free/open source facebook and twitter?

9. Alan says:[10 April 2010 8:45 PM](#2f3cff828375cb949e3b89c9bee66cd8a2b93aa4)

   Awesome! This is great stuff, Andy.

   Forgive me: I usually despise these sorts of vacuous comments. But this is so thrilling to me that I can't resist the temptation just this once.

10. [Keith Rarick](http://xph.us/) says:[10 April 2010 9:15 PM](#a1e02480cec7947bc5b7a16784138c5cb464d896)

    For what it's worth, I fully agree. It seems like most people (even those who recognize that a problem exists) don't quite grasp just how fundamental the problem is. Now, with iPhone OS, it looks like Apple wants to extend this problem even to devices that you own.

    Your proposal sounds good, but, as hurdles go, I fear that the technical ones are only the beginning. Human, non-technical, design appeal (of protocol, API, documentation) will be crucial. It must appeal to developers, especially those who understand viral marketing. Facebook has the "friend finder" and related tools for inviting other users (plus a thousand other, smaller, design choices). The biggest reason for their success is that they understand how important that is.

    Assuming success, this would be an amazing platform for other applications, too. Consider email, publishing, commerce, music, video. The potential benefit to society is hard to overestimate.

11. [Rory McCann](http://technomancy.org/) says:[10 April 2010 9:54 PM](#8586a06a505cc215f373cc478981fd606d3383eb)

    If Homomorphic Encypytion (http://en.wikipedia.org/wiki/Homomorphic\_encryption) becomes possible, then this will solve a lot of problems, since you can then store \*and\* process the data on someone else's server without revealing the data or giving them your private key.

    If homomorphic encyption becomes possible, then that changes one of your biggest assumptions and one will have to think out the possibilities.

12. fkooman says:[10 April 2010 10:33 PM](#1a1e2a5366e24a7b578777a578b63320e81bca0e)

    See also Eben Moglen's excellent Freedom in the cloud @ http://www.isoc-ny.org/?p=1338 which touches on some of the fundamental issues of cloud freedom and about hosting everything yourself.

13. [Pierre St Juste](http://www.socialvpn.org) says:[11 April 2010 2:43 AM](#380fa17577ae40c3430acfb68935cde1d8990221)

    Here's a quick solution

    SocialVPN + LAMP + Wordpress

    Direct P2P VPN with friends for communication (makes it look like your friends are part of a Social LAN)

    Run LAMP (for Linux) or WAMP (for windows)

    Install wordpress on LAMP/WAMP

    Final product = a locally installed wordpress blog that only your friends can access, your friendlist would be a blogroll pointing to other locally installed wordpress blogs.

    http://www.socialvpn.org

14. elima says:[11 April 2010 9:26 AM](#063067e4d103bc38226ce7b3a8d2e566ce92922d)

    Thanks for the great inside.

    For data storage, I think semantic web technologies (RDF, FOAF, etc), together with document oriented databases such as CouchDB or Mongo are the right place to look at. Flexibility in data schemes should be a major concern when designing storage, for it's almost impossible to grasp the huge variety of personal data one will want to federated from their owned nodes if such system gets implemented. Basically, all the information one person is authoritative for: profile info, photos, social relations, CV, weblog posts, status info, photos, books, ..., (name any person-oriented service out there).

    There is also another initiative towards federated (autonomous) social networking at Mediamatic Lab (http://www.mediamatic.net/page/26258/en), but info is quite disperse there.

15. Steve Baker says:[12 April 2010 3:49 AM](#df8c7c17a99d34d251f33a842236f1cb0cf5093b)

    Hi Andy

    I've been thinking about this problem a bit too. One option for the peer-to-peer distributed store in your diagram is the Google Wave Federation Protocol.

    http://www.waveprotocol.org/

    This runs over XMPP and allows wave servers to federate with each other.

    In a model where everyone ran their own wave server, a person could add their friends as participants in a piece of content and the federation protocol would ensure it propagates to the friends' servers and keeps all content in sync.

    Google have releases specs and an example Java server/api. Its up to us now to implement Operational Transformation in our pet languages (\*cough\*guile), and write frontends to provide the social networking experience we want.

    cheers

16. [Matija "hook" Šuklje](http://matija.suklje.name) says:[12 April 2010 9:07 AM](#5af7ec22ea4b9074cca79a10d80ac6843a0db624)

    It's not exactly the same, but you might want to check out OwnCloud as well. IMHO something like GNU Social or what you proposed + OwnCloud on a small plug computer in each geek's house would rock :\]

    http://owncloud.org

17. [Brian Gough](http://www.briangough.ukfsn.org/) says:[12 April 2010 3:55 PM](#224a84e59a504f0caf3750286a549caebf0a7f26)

    Just to clarify one point about RMS's "Software as a Service" article not going far enough, when he presented it at LibrePlanet he actually said the same thing---that there were many other potential problems (such as privacy) with websites that are not "Software as a Service" but that he wanted to urgently address the SaaS issue specifically because SaaS is even worse than proprietary software. I think he said he was working on a further article that would address the other issues.

    There should be a video of the talk available at some point where this will be clearer.

18. leighman says:[12 April 2010 9:15 PM](#f29ddeace778b7d6f94ac45364cd32d21e87eccb)

    Isn't this kinda what Opera Unite sets out to do?

    Other than Opera not being free =D

19. [jors](http://enchufado.com/) says:[13 April 2010 9:33 AM](#1381ffcbcbe16db8303f387e1b14353fe69b89d8)

    Good brainstorming, though as per other comments, many others thought before about this!

    I'm with #Steve: FB was the first successful social network, and it will be really difficult to move users from there to any other place. It's like the MP3 format: it was the first and it came to stay.

20. fanboy says:[13 April 2010 5:01 PM](#f391564bfc6d175a11a0b2deefb4df54ff778b1f)

    Why isn't there a mention of StatusNet (status.net) or OStatus (ostatus.org)

21. [Daeng Bo](http://blog.ibeentoubuntu.com) says:[13 April 2010 10:05 PM](#b712dcec150f49b214eb5caa8d8785f98130cbb8)

    Another vote for OneSocialWeb ( http://onesocialweb.org/ ). It's based on XMPP, open, federated, and works now. Apache license. It seeks to do the whole social thing, not just status updates.

22. skierpage says:[20 April 2010 2:26 AM](#25c65666f9c07d98553100577e1a862d90987e88)

    I want to run my social networking peer-to-peer from my computer, that way a) I can choose exactly which groups of associates I share photos and personal stuff with, b) I have a comprehensive local record of what I've shared and what's been shared with me (Clicking "Older posts" in FB six times then saving the resulting web page doesn't count ;-) ). Good luck making it happen.

    I'm not sure where you're going with the content-based store/git stuff. So long as I have access to my friend list, if I lose my local social network information, my local social network software can contact them all and say "Give me all the stuff skierpage shared with you and that you shared with skierpage", and they'll be able to reconstruct for me everything except stuff I never shared with anyone.

    BTW, the first successful social network was e-mail. RSS feeds of blog posts with traceback was another. Putting it all on someone's central server is a step backward!

23. [Thomas Kappler](http://jugglingbits.wordpress.com/) says:[27 April 2010 11:06 AM](#2328c5a82a5217a20021e5da2d770368e3981824)

    You'll probably see this on planet gnome, but Luis Villa posted about http://joindiaspora.com/ today, which seems to share these goals.

24. [Jiri Baum](http://www.baum.com.au/sabik) says:[5 May 2010 10:11 AM](#71469edb3ec829e6199a70439a1dd3febee6b5b0)

    My own attempt at this is called connectr; I started by paring down the Facebook concept to minimum useful functionality, getting Twitter, then implementing that first — the theory being that features can be added gradually, but minimum useful functionality is key.

    Currently I have about three or four regular users (including myself); growing that will be the real trick, of course.

    http://www.baum.com.au/connectr

25. Luuk de Waal Malefijt says:[13 May 2010 6:15 PM](#64250a42387f566461785160557a2ffc1384b784)

    Good read. You might want to check out the Diaspora and Appleseed projects too. Or join the discussion on Digg.

    http://digg.com/design/Diaspora\_The\_Student\_Made\_Privacy\_Respecting\_Facebook\_Alte

Comments are closed.
