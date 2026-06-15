---
title: biting the hand that feeds — wingolog
url: https://wingolog.org/archives/2008/03/07/biting-the-hand-that-feeds
published: "2008-03-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/03/07/biting-the-hand-that-feeds
---

# biting the hand that feeds — wingolog

## [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)

7 March 2008 2:26 PM

- [propagandhi](/tags/propagandhi)
- [music](/tags/music)
- [aikido](/tags/aikido)
- [tekuti](/tags/tekuti)
- [scheme](/tags/scheme)
- [git](/tags/git)
- [meta](/tags/meta)

**aikido**

In a couple hours I'll go up to the polideportivo and help lay out 500 square meters of mat. It's seminar weekend, kiddos! The incoming instructors are top-notch, [Yamada sensei](http://www.nyaikikai.com/yamada.asp) out of New York and Shibata sensei out of Hombu dojo in Tokyo. As a bonus we'll have [Peter Bernath](http://www.floridaaikikai.com/staff.htm), [Harvey Konigsberg](http://www.woodstockaikido.com/instructors.htm), and about 50 or so other people coming from abroad. Good times!

And, and, *para colmo*, tomorrow I test for black belt. Yay!

A lot of people ask what happens once you get your black belt. Its traditional meaning is that you are a serious student, and have an understanding of the basics of a martial art. It does not connote finality in any way; it's more like a milestone, or something like that.

One can see this in the first-degree tests, like mine tomorrow. They're usually fun to watch, but unnecessarily forced, lacking in grace. The difference between first- and second-degree tests is phenomenal, though -- it seems that in the few years after shodan, practitioners gain a sense of confidence and fluidity that they lacked before. That I lack now, I mean. So it's an important rite, for me, but one that points towards the future rather than the past.

**punk**

The album "Less Talk, More Rock" by Propagandhi is a near-masterpiece. While I do like their other albums, "Less Talk, More Rock" has an infectious youthful brilliance that makes me twitch every time I hear it. I must have listened to *Resisting Tyrannical Government* 50 times and it is still a transformative experience. Rock on!

**tekuti**

Since [last week's missive](//wingolog.org/archives/2008/02/29/im-telling-you/), I've been able to relax a bit, hack-wise, fixing errors as I see them. Most errors have been related to the fact that displaying a blog entry first parses it as valid XML, throwing an exception if the input is invalid. Luckily wordpress is pretty good at ensuring that its text is valid XML, but it's not complete -- it allows bare ampersands, both in the text and in URLs, and sometimes lets angle-brackets pass through unfiltered. So I've had to fix up a few old posts.

Among the more curious things I have had to write for this blagware is a UTF-8 encoder, in order to parse character references like `’` and such, given that Guile only does byte strings, currently.

Shockingly, to me, I do get spam, on the order of about one or two comments per day. *No one else uses this software.* It seems that there are a couple bots out there that actually parse forms, looking for textareas, then manage to divine which fields require what syntax. Currently my field names are the same as wordpress', so I will vary them until my obscurity provides the necessary "security".

But in the meantime, since my persistent store is Git and not a database, I can easily revert any change, be it changes to posts or to comments or whatever. I fleshed out the admin interface sufficiently so that you can actually create and edit posts there, and gave it an interface for seeing recent changes and possibly reverting them. Of course, reversion is also a change which can be reverted, ad infinitum, so there is no need for scary warnings in the UI when deleting comments, because no change is irreversible. Neat.

## related articles

- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [meta data](/archives/2010/12/13/meta-data)
- [metablog](/archives/2008/04/23/metablog)
- [git: a transcriptional database](/archives/2008/04/12/git-a-transcriptional-database)
- [I'm telling you](/archives/2008/02/29/im-telling-you)

Comments are closed.
