---
title: code not burgers — wingolog
url: https://wingolog.org/archives/2010/04/05/code-not-burgers
published: "2010-04-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/04/05/code-not-burgers
---

# code not burgers — wingolog

## [code not burgers](/archives/2010/04/05/code-not-burgers)

5 April 2010 6:45 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)
- [summer of code](/tags/summer%20of%20code)
- [soc](/tags/soc)
- [google](/tags/google)

I know your problem: you are a student, and the Google Summer of Code has started, but you haven't found a project worth doing. Or, perhaps you have submitted a proposal already, but you're not psyched about it. You really wanted to munge bytes on disk using Scheme. I totally dig it.

That is why you should make a proposal to make a library for accessing Git datastores from Guile. Great idea, right? You probably had it too!

The scope of the work would be to write the equivalent of Git's "plumbing" layer in Scheme. The first step would be to be able to git cat-file any object in the database. Then you would need to be able to insert into the database, via git-hash-object, and then to be able to update a ref with git-update-ref.

At the same time, all of this functionality should be available on the command line, via something like a `gilt` executable: `gilt cat-file foo`, for example.

The reason why this work is interesting is N-fold:

- Having a Git library will be nice. Indeed [a number of software projects](https://git.wiki.kernel.org/index.php/Interfaces,_frontends,_and_tools) use Git as a database.

- Having a Git implementation that can be linked to Guile will be good too. (Guile is LGPLv3+, Git is GPLv2 only.)

- It will be interesting to see how fast a Scheme-only solution will be. Raw SHA1 calculation will probably still have to be done in C though.

- Finally it's going to be total fun! It's a good scope for an SOC project: you can implement as far as you get, and you learn skills like how to do real-world development in Scheme, producing fast code.

Interested? Time is running out. 9 April appears to be the deadline. Requirements are a proficiency with Git, and you should implement a command-line Guile program that returns the MD5 or SHA-1 hash of the file given as its argument. It's OK to copy the MD5 or SHA-1 implementation from elsewhere, but document that; I just want to know if you can actually wire up the pieces correctly, and beyond that, is your Scheme in good taste.

So, should this thing be up your alley, send me a mail, or visit #guile during the European daytime, and let's talk!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

### 3 responses

1. Benjamin Otte says:[6 April 2010 0:04 AM](#ee148cf804d6a178ac8bb587523453857e727ed7)

   s/gilt/guilt/

2. [wingo](http://wingolog.org/) says:[6 April 2010 7:22 AM](#bd902cf19268e9fe17d1dc760cabb895ce29614a)

   Are you goading me to ungild? Perhaps geld would be a better name, then.

   :)

3. [Samium Gromoff](http://blog.feelingofgreen.ru) says:[6 April 2010 2:08 PM](#bc1e8c2f72f287646930a87394101de0cedffb2f)

   I've been contemplating writing a plumbing layer in Common Lisp for quite some time already.

   The benefit over guile, is, of course, native compilation.

   If ported to ECL\[1\], this could be actually be made linkable to C programs, as it is capable of producing .so/.dll files.

   1\. Embeddable Common Lisp, ecls.sourceforge.net

Comments are closed.
