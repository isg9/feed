---
title: eeeevil — wingolog
url: https://wingolog.org/archives/2007/11/28/eeeevil
published: "2007-11-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/11/28/eeeevil
---

# eeeevil — wingolog

## [eeeevil](/archives/2007/11/28/eeeevil)

28 November 2007 8:25 PM

- [python](/tags/python)
- [evil](/tags/evil)

Just now, I wanted to define the `all` function in Python, which takes a sequence as its argument and returns `True` iff no element of the sequence is False. My instinct was to do it with `reduce`:

```
all = lambda seq: reduce(and, seq, True)

```

But `and` is a syntactic keyword, so that doesn't work. However, abusing the fact that [Python follows Iverson's convention](http://drj11.wordpress.com/2007/05/25/iversons-convention-or-what-is-the-value-of-x-y/), we can map `int.__mul__` instead:

```
all = lambda seq: reduce(int.__mul__, seq, True)

```

Evil, delicious evil. Do check out [David Jones' weblog](http://drj11.wordpress.com/) if you haven't already, it's really interesting. I started reading it the other day and couldn't stop :)

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)
- [object closure and the negative specification](/archives/2008/04/22/object-closure-and-the-negative-specification)
- [reducing the footprint of python applications](/archives/2007/11/27/reducing-the-footprint-of-python-applications)

### 9 responses

1. duff says:[28 November 2007 8:56 PM](#3184)

   A more readable approach would be to use "operator.and\_".

2. [wingo](http://wingolog.org/) says:[28 November 2007 9:03 PM](#3186)

   duff: You speak the truth. I'm just amused that you can multiply by False.

3. [Rob Bradford](http://www.robster.org.uk) says:[28 November 2007 9:05 PM](#3187)

   Or, better:

   all = lambda seq: reduce(bool.\_\_and\_\_, seq, True)

   This is better because it returns True or False not 0 or 1.

4. Phil says:[28 November 2007 9:11 PM](#3188)

   FYI, all is built-in in Python 2.5.

5. Josh says:[28 November 2007 11:07 PM](#3193)

   all = lambda seq: False not in seq

6. Johan Dahlin says:[28 November 2007 11:21 PM](#3194)

   Josh: or rather,

   all = lambda seq: False not in map(bool, seq)

7. Josh says:[28 November 2007 11:48 PM](#3195)

   Johan,

   If you want the same behavior as the built-in all function, you are correct. Andy's requirement "returns True iff no element of the sequence is False" is not clear as to whether he means False or anything that converts to False.

   built-in: all((True, None)) returns False

   It could be argued that no element in that sequence is "False". My solution would return True. I could not think of a simple way around 0 == False. i.e. my all((True, 0)) returns False like yours.

8. [Pádraig Brady](http://www.pixelbeat.org/) says:[29 November 2007 3:25 PM](#3206)

   Johan, you want 'itertools.imap' instead of 'map'

   to have the same functionality as the builtin all()

   available since python 2.5.

   I've been playing with that here:

   http://www.pixelbeat.org/scripts/funcpy

   Andy, Your's is the only site I've noticed that I

   can't connect to from home. Though I can connect to

   it from work?

9. [wingo](http://wingolog.org/) says:[29 November 2007 7:18 PM](#3210)

   Pádraig: Hm, I don't know anything about that. If you get a chance, can you send me a mail (wingo pobox com) with the traceroute from home?

Comments are closed.
