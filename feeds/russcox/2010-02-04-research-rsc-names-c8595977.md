---
title: 'research!rsc: Names'
url: https://research.swtch.com/names
published: "2010-02-04T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/names
---

# research!rsc: Names

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Names Russ Cox February 4, 2010 *research.swtch.com/names* Posted on Thursday, February 4, 2010.

Every programmer has a variable naming philosophy. This is mine:

> *A name's length should not exceed its information content.* For a local variable, the name `i` conveys as much information as `index` or `idx` and is quicker to read. Similarly, `i` and `j` are a better pair of names for index variables than `i1` and `i2` (or, worse, `index1` and `index2`), because they are easier to tell apart when skimming the program. Global names must convey relatively more information, because they appear in a larger variety of contexts. Even so, a short, precise name can say more than a long-winded one: compare [`acquire`](http://www.google.com/codesearch?q=acquire) and [`take_ownership`](http://www.google.com/codesearch?q=take_?ownership). Make every name [tell.](http://www.bartleby.com/141/strunk5.html#13)

The information content metric gives a quantitative argument against long-winded names: they're simply inefficient. I internalized this metric years ago but only realized this phrasing of it recently, perhaps because I have been looking at [too](http://www.google.com/codesearch?q=getParametersAsNamedValuePairArray) [much](http://www.google.com/codesearch?q=org\.apache\.commons\.httpclient\.MultiThreadedHttpConnectionManager) [Java](http://www.google.com/codesearch?q=com\.google\.common\.base\.FinalizablePhantomReference) [code](http://www.google.com/codesearch?q=).

(Comments originally posted via Blogger.)

- [Barry Kelly](http://www.blogger.com/profile/10559947643606684495)(February 4, 2010 9:41 AM) That's all very well, but what **would** you rename FinalizablePhantomReference to?

I can see arguments for shortening Reference to Ref, but not much for shortening the rest of it.

- [Rob Kohr](http://www.blogger.com/profile/14011841852767765694)(February 4, 2010 10:43 AM) Nonsense.

Short names are useless for readable code. I would prefer some long winded name that explains what it is for rather than "j"

What is j? How do I know what j is used for?

I could see if j is just a simple incrementer for a loop, but if it actually has any meaning beyond that, it should have a name to denote that meaning.

- [David](http://www.blogger.com/profile/05047921612824798958)(February 4, 2010 12:40 PM) I also think this is nonsense. This makes code much less readable. You may be able to understand your variable names, but others will have to infer into the code to determine what the variable does. A good descriptive name will eliminate this step and allow the reader to know what the variable does before having to sort through its use.

- [Ilyia Kaushansky](http://www.blogger.com/profile/05259373019890029073)(February 4, 2010 1:39 PM) The following seems to be relevant to the discussion:

http://www.artima.com/weblogs/viewpost.jsp?thread=266543

- [dmolony](http://www.blogger.com/profile/03671968110694943133)(February 4, 2010 1:42 PM) Rob and David appear to be postersWhoMissedThePointCompletely, but a more succinct variable name would be 'bozos'

- [Justin Lilly](http://www.blogger.com/profile/00756672514026667550)(February 4, 2010 2:07 PM) Long variable names are only "inefficient" if you have to make up for your editor's misgivings. An editor worth its salt won't make you type more than the first 3 letters.

- [David Andersen](http://www.blogger.com/profile/03996590425188586871)(February 4, 2010 2:37 PM) A few comments:

Rob: You wrote, "I could see if j is just a simple incrementer for a loop, ". Russ wrote: "Similarly, i and j are a better pair of names for index variables." q.e.d.

Russ: I think that there's actually room for a sliding scale here that you allude to in your philosophy about global variables. My view on this: *The descriptiveness of a variable name should be directly related to the size of the scope in which that name can be used.*. Thus, a member variable of a widely-used struct can probably benefit from a slightly more descriptive name than an index variable, etc.

Justin: A lot of "good" programming practices aim to ensure that the information a programmer needs in order to understand the code is mostly in front of them -- on the same screen, within a few lines, etc. Short variable names can hurt this (by requiring a reference to where they were declared or commented or initialized), or improve this (by keeping the code short so that more of it is in context). Russ's point, as I interpret it, is that there's a good way to achieve a balance of this

- [Russ Cox](http://swtch.com/~rsc/)(February 4, 2010 4:54 PM) Dave is exactly right.

1\. The information content you need depends on the scope across which the variable will be used. I agree 100% with Dave's italicized rule.

2\. Once you've decided how descriptive the name must be (how much information to put in), don't settle for a low-density encoding if there is a high-density one that is at least as readable.

For example, one of the links is to a function named getParametersAsNamedValuePairArray. The only interesting word in that whole name is "parameters", because it's clear from the function signature that you get them and that they come back as an array of name value pairs. And note the inaccuracy: a named value pair would be (name, value1, value2) but the array element is actually a name value pair (name, value). Not only is the name verbose, it's wrong. So parameters() or params() would be just as good, and more accurate, than getParametersAsNamedValuePairArray. With the extra space you've saved, you might even add information. For example, you could say which parameters, as in queryParams().

- [Kai](http://www.blogger.com/profile/07926438495948801306)(February 19, 2010 10:06 PM) This short note by Chris Seiwald echoes your policy on naming. Interesting read in any case:

http://www.perforce.com/perforce/papers/prettycode.html

- [J](http://www.blogger.com/profile/16615240170850222143)(July 3, 2010 11:56 AM) Yep, I love the way rio, acme - damn, the whole system, is written!
