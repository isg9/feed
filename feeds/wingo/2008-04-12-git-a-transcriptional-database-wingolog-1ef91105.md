---
title: 'git: a transcriptional database — wingolog'
url: https://wingolog.org/archives/2008/04/12/git-a-transcriptional-database
published: "2008-04-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/12/git-a-transcriptional-database
---

# git: a transcriptional database — wingolog

## [git: a transcriptional database](/archives/2008/04/12/git-a-transcriptional-database)

12 April 2008 7:11 PM

- [git](/tags/git)
- [poetry](/tags/poetry)
- [khayyam](/tags/khayyam)
- [translation](/tags/translation)
- [scheme](/tags/scheme)
- [tekuti](/tags/tekuti)

I had a thought yesterday while biking into town. [Git](http://git.or.cz/) is a *transcriptional* database -- it writes and writes and writes, and what we are left with is its transcript. I wouldn't call it a transactional database, since it has no rollback operator. It doesn't need one. Ref updates either succeed or fail. If they fail, well, write, write again:

> The Moving Ref writes; and, having commit,
>
> Moves on: cosmic Rays nor Zero-blit
>
> Shall untrue its blobs, its trees Unbind,
>
> Nor all your Pushes flip a Single bit.

It is perhaps not as beautiful as [Fitzgerald's translation](http://www.cs.rice.edu/~ssiyer/minstrels/poems/545.html), but the bar was set quite high. To compensate, here is what is, to my knowledge, the first translation of Khayyam into Scheme:

```
(define (git-update-ref refname proc count)
  (let* ((ref (git-rev-parse refname))
         (commit (proc ref)))
    (cond
     ((zero? count) #f) ; failure
     ((false-if-git-error
         (git "update-ref" refname commit ref))
      commit)
     (else
      (git-update-ref (git-rev-parse refname) (1- count))))))

```

The rest of the text may be found in [tekuti](//wingolog.org/software/tekuti/).

## related articles

- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)
- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [for love and $](/archives/2012/02/21/for-love-and-)
- [meta data](/archives/2010/12/13/meta-data)

### 4 responses

1. [bartman](http://www.jukei.net) says:[12 April 2008 10:10 PM](#0b31d3074eb14cba57c351ff395c109aa8cf8e6c)

   No rollback? How about git checkout or git reset? Both of those roll back some subset of changes depending on arguments.

2. [wingo](http://wingolog.org/) says:[12 April 2008 10:18 PM](#92ae37b18c071a3d680e0df73c6c434f07589f4c)

   Neither checkout nor reset affect the database. "Checkout" affects the ephemeral "working tree", one instance of an area in which to stage changes. "Reset" changes the values of ref pointers. But as far as the object database goes, all writing is on clean sheets of paper: nothing is ever overwritten.

3. [ulrik](http://users.student.lth.se/f04us/) says:[13 April 2008 9:14 AM](#1b4e8e0116780088c8d5cf0c3fac9c8e73fb8fdd)

   I must be a real git nut, no even totally nutty since I was thinking about starting a comment with "Git speaks to us and tells us.."

   Anyway what I think that was about is the question why git's internals are so interesting and those of other scms not. It seems that git's internals are very well designed and indeed I believe Linus now he claims "Design your datastructures well and the rest will be natural".

   I like git a lot. What it tells us is that the end result is uncomplicated and getting there is just a number of small steps, each of them pretty simple. Then on top of that simplicity you can build magic like git-blame or git-repack.

4. [wingo](http://wingolog.org/) says:[13 April 2008 10:37 AM](#82a0f872bda2763d29256848f3d78bc1e2c43463)

   Ulrik, even if you're a nut, you're in good company :)

Comments are closed.
