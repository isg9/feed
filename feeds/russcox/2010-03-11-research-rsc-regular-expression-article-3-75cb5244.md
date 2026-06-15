---
title: 'research!rsc: Regular Expression Article #3'
url: https://research.swtch.com/regexp3
published: "2010-03-11T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/regexp3
---

# research!rsc: Regular Expression Article #3

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Regular Expression Article \#3 Russ Cox March 11, 2010 *research.swtch.com/regexp3* Posted on Thursday, March 11, 2010.

In January 2007 I posted an article on my web site titled “ [Regular Expression Matching Can Be Simple And Fast.](http://swtch.com/~rsc/regexp/regexp1.html)” I intended this to be the first of three; the second would explain how to do submatching using automata, and the third would explain how to make a really fast DFA. I posted the [second article](http://swtch.com/~rsc/regexp/regexp2.html) a few months ago.

Today, the [third and final article](http://swtch.com/~rsc/regexp/regexp3.html) is available, along with an open source production implementation called [RE2](http://code.google.com/p/re2).

(Comments originally posted via Blogger.)

- [tef](http://www.blogger.com/profile/02751385132475323535)(March 11, 2010 12:04 PM) Hi, the following images are a 404:

http://swtch.com/~rsc/regexp/cat\_Lu.png

http://swtch.com/~rsc/regexp/script\_Greek.png

linked in the sentence: " for example, look at \\p{Greek} (the Greek script) or at \\p{Lu} "

- [Russ Cox](http://swtch.com/~rsc/)(March 11, 2010 12:33 PM) @tef: Fixed, thank you.

- [niemeyer](https://launchpad.net/%7Eniemeyer)(July 7, 2010 3:55 PM) It looks very nice, thanks for publishing this.

Do you plan to evolve Go's regexp package in this direction as well?
