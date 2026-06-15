---
title: 'research!rsc: Regular Expression Article #2'
url: https://research.swtch.com/regexp2
published: "2009-12-10T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/regexp2
---

# research!rsc: Regular Expression Article #2

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Regular Expression Article \#2 Russ Cox December 10, 2009 *research.swtch.com/regexp2* Posted on Thursday, December 10, 2009.

In January 2007 I posted an article on my web site titled “Regular Expression Matching Can Be Simple And Fast.” I intended this to be the first of three; the second would explain how to do submatching using automata, and the third would explain how to make a really fast DFA. I got distracted for a few years but have finally finished the second article.

[http://swtch.com/~rsc/regexp/regexp2.html](http://swtch.com/~rsc/regexp/regexp2.html)

Simple (easy to follow, I hope) code demonstrating the techniques from the article can be found at [http://code.google.com/p/re1/](http://code.google.com/p/re1).

(Comments originally posted via Blogger.)

- [Tobias Svensson](http://www.blogger.com/profile/14182007029531911943)(December 10, 2009 12:06 PM) Great write up! I'm happy to see that you're posting regularly again, as I have always greatly enjoyed your blog.

- [tim.henderson](http://www.blogger.com/profile/07228760916156974582)(December 10, 2009 7:57 PM) I hope you do write your third post, I found your first two to be among the most readable explanations on this topic I have encountered.

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(December 11, 2009 6:17 AM) beautifully clear article, thanks.

BTW, i think this paragraph might still be in its nascent stage: "If have other criteria can compare submatch sets and choose the better one. The Eighth Edition Unix library used leftmost longest to match DFA-based tools like awk and egrep."

- [Marius Gedminas](http://www.blogger.com/profile/15155998626202067226)(December 11, 2009 6:41 AM) A very interesting article, thank you!

One note: "Character classes are typically implemented as a special VM instruction rather than expanding them into alternations, just as we did for dot above." refers to something that doesn't exist in the text.

- [Russ Cox](http://swtch.com/~rsc/)(December 11, 2009 8:30 AM) Thanks everyone; I've corrected these mistakes and a couple others.

I also added a bit more about POSIX submatching; thanks to Chris Kuklewicz.

- [Matt Doar](http://www.blogger.com/profile/02360651363519410698)(December 15, 2009 9:56 AM) The article is the best around - well written, obviously carefully edited. Great job! Looking forward to #3

~Matt

- [Jonathan](http://www.blogger.com/profile/05317757243529974467)(December 28, 2009 12:58 AM) Thanks for the article about little virtual machines.

A few weeks back I implemented a little virtual machine for a processing task. The close resemblance of the two virtual machines makes me feel vindicated.
