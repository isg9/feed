---
title: it's probably spam — wingolog
url: https://wingolog.org/archives/2017/03/06/its-probably-spam
published: "2017-03-06T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2017/03/06/its-probably-spam
---

# it's probably spam — wingolog

## [it's probably spam](/archives/2017/03/06/its-probably-spam)

6 March 2017 2:16 PM

- [spam](/tags/spam)
- [tekuti](/tags/tekuti)
- [meta](/tags/meta)
- [bayesian](/tags/bayesian)
- [classification](/tags/classification)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [hubris](/tags/hubris)

Greetings, peoples. As you probably know, these words are served to you by [Tekuti](https://wingolog.org/software/tekuti/), a blog engine written in Scheme that uses Git as its database.

Part of the reason I wrote this blog software was that from the time when I was using Wordpress, I actually appreciated the comments that I would get. Sometimes nice folks visit this blog and comment with information that I find really interesting, and I thought it would be a shame if I had to disable those entirely.

But allowing users to add things to your site is tricky. There are all kinds of potential security vulnerabilities. I thought about the ones that were important to me, back in 2008 when I wrote Tekuti, and I thought I did a pretty OK job on preventing XSS and designing-out code execution possibilities. When it came to bogus comments though, things worked well enough for the time. Tekuti uses Git as a log-structured database, and so to delete a comment, you just revert the change that added the comment. I added a little security question ("what's your favorite number?"; any number worked) to prevent wordpress spammers from hitting me, and I was good to go.

![](https://wingolog.org/pub/spam-attack.png)

Sadly, what was good enough in 2008 isn't good enough in 2017. In 2017 alone, some 2000 bogus comments made it through. So I took comments offline and painstakingly went through and separated the wheat from the chaff while pondering what to do next.

**an aside**

I really wondered why spammers bothered though. I mean, I added the [`rel="external nofollow"`](https://www.mattcutts.com/blog/?s=nofollow) attribute on links, which should prevent search engines from granting relevancy to the spammer's links, so what gives? Could be that all the advice from the mid-2000s regarding `nofollow` is bogus. But it was definitely the case that while I was adding the attribute to commenter's home page links, I wasn't adding it to links *in* the comment. Doh! With this fixed, perhaps I will just have to deal with the spammers I have and not even more spammers in the future.

**i digress**

I started by simply changing my security question to require a number in a certain range. No dice; bogus comments still got through. I changed the range; could it be the numbers they were using were already in range? Again the bogosity continued undaunted.

So I decided to break down and write a bogus comment filter. Luckily, Git gives me a handy corpus of legit and bogus comments: all the comments that remain live are legit, and all that were ever added but are no longer live are bogus. I wrote a [simple tokenizer](https://gitlab.com/wingo/tekuti/blob/master/tekuti/classifier.scm#L40) across the comments, [extracted feature counts](https://gitlab.com/wingo/tekuti/blob/master/tekuti/classifier.scm#L67), and fed that into a [naive Bayesian classifier](https://gitlab.com/wingo/tekuti/blob/master/tekuti/classifier.scm#L104). I finally turned it on this morning; fingers crossed!

My trials at home show that if you train the classifier on half the data set (around 5300 bogus comments and 1900 legit comments) and then run it against the other half, I get about 6% false negatives and 1% false positives. The feature extractor interns sequences of 1, 2, and 3 tokens, and doesn't have a lower limit for number of features extracted -- a feature seen only once in bogus comments and never in legit comments is a fairly strong bogosity signal; as you have to make up the denominator in that case, I set it to indicate that such a feature is 99.9% bogus. A corresponding single feature in the legit set without appearance in the bogus set is 99% legit.

Of course with this strong of a bias towards precise features of the training set, if you run the classifier against its own training set, it produces no false positives and only 0.3% false negatives, some of which were simply reverted duplicate comments.

It wasn't straightforward to get these results out of a Bayesian classifier. The "smoothing" factor that you add to both numerator and denominator was tricky, as I mentioned above. Getting a useful tokenization was tricky. And the final trick was even trickier: limiting the significant-feature count when determining bogosity. I hate to cite Paul Graham but I have to do so here -- choosing the N most significant features in the document made the classification much less sensitive to the varying lengths of legit and bogus comments, and less sensitive to inclusions of verbatim texts from other comments.

We'll see I guess. If your comment gets caught by my filters, let me know -- over email or Twitter I guess, since you might not be able to comment! I hope to be able to keep comments open; I've learned a lot from yall over the years.

## related articles

- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [meta data](/archives/2010/12/13/meta-data)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [for love and $](/archives/2012/02/21/for-love-and-)
- [metablog](/archives/2008/04/23/metablog)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)

### 6 responses

1. [hendi](http://hendi.io) says:[6 March 2017 2:52 PM](#af594bd782323b4a00514355d66511786af4f9ee)

   Interesting writeup! Years ago when I still used Evolution I played around with spamassassin, bogofilter and my own naive implementation of a Bayesian filter. Needless to say my approach performed much worse than the more established contenders, but it provided a great learning experience. Similar to your discovery now I've also learned that limiting the calculation to the N most significant features improved the accuracy tremendously.

   I'm checking out Tekuti's source right now, I'm intrigued that it uses git as a database/log storage and I love to dig through some Scheme/Guile code anyway.

   Btw, do you need some VIAGRA or a [FREE CREDIT CARD LOAN](https://rickroll.me)? ;p

2. [wingo](http://wingolog.org) says:[6 March 2017 2:57 PM](#32d33aa8241539df5ffa53680f484189a12b322b)

   Thanks for the note, Hendi, though no thanks for poisoning my classifier ;) I admit, part of the motivation here was just to see how a classifier worked. I guess one day, hopefully far in the future, I'll have to learn about neural networks instead :)

3. [Alex Reinhart](https://www.refsmmat.com/) says:[6 March 2017 3:09 PM](#c72aa9e1f775453cfbc3ee7ed18afba6639f1238)

   Speaking from experience running a large forum, rel=nofollow has limited effect. Same with random questions and answers. Same with reCAPTCHA, actually. I suspect registration is outsourced to real humans who get paid a few cents to beat the CAPTCHA and registration checks, and I also suspect the motive is no longer just SEO.

   There's an interesting article out this morning, for example, which suggests one of the major email spam organizations makes money by selling genuine display advertising to companies, then sending spam to drive people to click the ads. So they are interested in your real human viewers, not the search engine robots. There's probably another contingent interested in spreading malware, another in scamming people, another in selling them real but shady products...

   http://www.csoonline.com/article/3176433/security/spammers-expose-their-entire-operation-through-bad-backups.html

   At this point I'm ready to start charging people $5 to register. That's probably the only option left.

4. [John Cowan](http://vrici.lojban.org/~cowan) says:[6 March 2017 3:18 PM](#7a906995fa89bdb0dcde8acd0095fd0aac289e22)

   You could convert each comment to an email and send it to a Gmail address you control. Whatever survives Google's aggressive spam filtering can be pulled back using POP3 or IMAP.

5. [Francesco](http://www.ariis.it) says:[6 March 2017 3:39 PM](#9b7581dfb819e1a31c243e4de7432d65a91c903f)

   Thanks for taking the pain of going through this and not just disabling comments.

   More and more blogs are following the latter path, subcontracting comments to Facebook and Twitter: understandable, but yet another victory for social walled gardens I loathe so much.

6. [Wilfred](http://wilfred.me.uk) says:[21 June 2017 5:58 PM](#283dfe3134e9b5b0bba610d659e9a7d7a11aecbe)

   Ouch, looks like some are still getting through. Have you considered just not allowing users to provide their own website?

Comments are closed.
