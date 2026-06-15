---
title: Trust Boundaries in Software Systems
url: https://blog.regehr.org/archives/1576
published: "2018-02-12T20:51:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=1576
---

# Trust Boundaries in Software Systems

One of the big things that has changed in computer science education over the last 20 years is that it is now mandatory to prepare students for writing software that lives in a hostile environment. This content can’t be limited to a computer security course, it has to be spread throughout the curriculum. My experience, based on talking to people, looking through textbooks, and looking at lecture material on the web, is that overall we’re not yet doing a great job at this.

When teaching this subject, I’ve started using trust boundaries as an organizing principle. A trust boundary exists any time we (the system designers or system owners) trust code, data, or human actors on one side of an interface more than we trust the other side of the interface. Students need to be able to recognize, understand, fortify, and stress-test the trust boundaries in any system they have a stake in.

Trust boundaries aren’t hard to find: We just need to ask questions like “What are the consequences if this code/data became horribly malicious? Is that likely? Can we defend against it? Do we want to defend against it?” It is easy to conclude, for example, that a demonic garbage collector or OS kernel might not be something that we wish to defend against, but that we had better fortify our systems against toxic PNG files that we load from random web sites.

Some basic observations about trust boundaries:

1. They’re everywhere, even inside code written by a single person. Anytime I put an assertion into my code, it’s a tacit acknowledgment that I don’t have complete trust that the property being asserted actually holds.

2. The seriousness of trust boundaries varies greatly, from mild mistrust within a software library all the way to major safety issues where a power plant connects to the internet.

3. They change over time: a lot of our security woes stem from trust boundaries becoming more serious than they had been in the past. Email was not designed for security. The NSA wasn’t ready for Snowden. Embedded control systems weren’t intended to be networked. Libraries for decoding images, movies, and other compressed file formats that were developed in the 90s were not ready for the kinds of creative exploits that they faced later on.

4. If you fail to recognize and properly fortify an important trust boundary, it is very likely that someone else will recognize it and then exploit it.

To deal with trust boundaries, we have all the usual techniques and organizing principles: input sanitization, defense in depth, sandboxing, secure authentication, least privilege, etc. The issue that I’m trying to respond to with this post is that, in my experience, it doesn’t really work to hand students these tools without some sort of framework they can use to help figure out where and when to deploy the different defenses. I’d be interested to hear how other CS instructors are dealing with these issues.
