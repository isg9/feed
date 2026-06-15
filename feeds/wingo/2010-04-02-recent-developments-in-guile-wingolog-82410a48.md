---
title: recent developments in guile — wingolog
url: https://wingolog.org/archives/2010/04/02/recent-developments-in-guile
published: "2010-04-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/04/02/recent-developments-in-guile
---

# recent developments in guile — wingolog

## [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)

2 April 2010 9:45 AM

- [gnu](/tags/gnu)
- [guile](/tags/guile)
- [ghm](/tags/ghm)
- [scheme](/tags/scheme)
- [emacs](/tags/emacs)

So, back in November I went to a [GNU Hackers Meeting](http://www.gnu.org/ghm/2009/) in Sweden. In December I wrote about part of what I got from it, [the relationship between GNU and the FSF](//wingolog.org/archives/2009/12/13/gnu-gnome-and-the-fsf). That post was long enough, so I left the rest for later, and later is finally now. So before heading out for the weekend, let me get to one of the pending topics.

**recent developments in guile**

I, your somewhat humble author, gave a talk! I spoke about recent developments in [GNU Guile](http://www.gnu.org/software/guile/), eventually discussing sneaky plans to get Guile into Emacs.

Here is my first foray into the world of ``:

If that doesn't work for you, you can download the video from a [torrent](http://www.gnu.org/ghm/2009/torrents/ghm2009-wingo-small.ogg.torrent), play the ogg from a [direct link](http://www.archive.org/download/ghm2009-wingo-small/ghm2009-wingo-small.ogg), or check the [archive.org page](http://www.archive.org/details/ghm2009-wingo-small) for other versions, including [H.264](http://www.archive.org/download/ghm2009-wingo-small/ghm2009-wingo-small_512kb.mp4), and even [animated GIF](http://www.archive.org/download/ghm2009-wingo-small/ghm2009-wingo-small.gif) (what?). Or download the [slides](//wingolog.org/pub/ghm-2009-guile-gnu-and-you.pdf), and save yourself my witticisms.

Since that talk, things have changed somewhat: Guile does have a dynamic foreign function interface, delimited continuations, eval implemented in Scheme, etc. It's unclear whether this next release will be 2.0, or the one after that; but I'm through with features.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the user in the loop](/archives/2011/10/19/the-user-in-the-loop)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 9 responses

1. didi says:[4 April 2010 1:08 AM](#1e59a0e99061f05e732541d69485abca755902cc)

   Nice talk. You got me excited about guile. But some questions still wander in my mind.

   Is guile a standalone language? I mean, should I write whole programs in it? Is it a extension language? The heavy lifting is done by C and the rest, by guile?

   I know what the site and history says about it being a extension language but it seems that you are going after a more independent approach.

2. [wingo](http://wingolog.org/) says:[5 April 2010 6:48 PM](#3262b9d06b9c03c87d1c82902123712fbf66ae46)

   Thanks for the note, didi. Guile works well as an extension language, but I personally have only used it that way when I first started with Guile, writing a custom Gnucash report. Since then all of my programs have been standalone Guile programs. I have written a number of C extensions to Guile, though.

   Hope that answers your question. Let me know if it didn't, though :)

3. Jeremy Banks says:[15 April 2010 5:56 PM](#d5b37a057145abf55ba74659f77f3f1610c23e56)

   Thanks for the interesting presentation. I didn't know much about Guile beforehand and now it sounds very appealing. Scheme and Javascript are both languages that I'd like to become more familiar with and it looks like Guile may be a wonderful environment in which to do so.

4. [Thomas Kappler](https://jugglingbits.wordpress.com) says:[19 April 2010 8:39 AM](#8f79f0f9f6f3b35ee986abd97d156f6d5876812f)

   Great talk, and great work on Guile! I finally got around to cleaning up some notes I took and blogging them. Please let me know in case I got anything wrong!

   https://jugglingbits.wordpress.com/2010/04/19/andy-wingo-recent-developments-in-guile/

5. Dmitry Dzhus says:[21 April 2010 8:20 PM](#ea50c6759887c363633050c83a22fc9493da9982)

   Guile is so cool, I'm happy it's rocking again. Keep it up and thank you for hacking for GNU!

6. Murat Demirtas says:[20 June 2011 10:52 PM](#c89a4859b979c5e3df5395c430651b9d3040414f)

   Hello Mister Wingo,

   I have been an observer at some time on guile. I have also compile guile 2.0.1 under linux. In windows its a little tricky. I have some question to you. You write you use Guile as standalone apllication. Are there some examples for standalone apps. As far as I have seen in the examples there are only C programs that can be expanded with guile. Is it possible to use it with c++. I use heaviliy qt4 framework and think about to make an attemp to use it in c++/qt4.

   Thanx in advance

7. yag says:[21 December 2011 8:10 PM](#f82be23b6b059c3f06ebcbf6fb30da0383f8de4f)

   I wonder, How far emacs integration has come.?

8. [Arne Babenhauserheide](http://draketo.de) says:[17 January 2013 6:06 PM](#929eb3c3dc7ba624755f31a7252174b5d32f1fdb)

   I just listened to your talk and it got me to experiment with tail call recursion again - together with a colleague.

   Thank you very much!

9. [zxq9](http://zxq9.com) says:[28 January 2013 7:27 PM](#35079656cd2036ae67965d4eff7c8768c12fb706)

   A bit late to the party... its the beginning of 2013 now.

   Guile 2 is out and we're scripting in it already. Emacs 23 is even more fun than Emacs 22 was -- but still no Guile implementation of elisp to be seen. Any updates on this?

   (Obviously its impossible to declare an effort like this dead, ever. The first mail I saw on this was dated 1998 or 1999. That says a lot -- not sure good or bad and not sure exactly about what its saying, but its interesting.)

Comments are closed.
