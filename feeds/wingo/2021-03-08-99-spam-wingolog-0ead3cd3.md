---
title: 99% spam — wingolog
url: https://wingolog.org/archives/2021/03/08/99-spam
published: "2021-03-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2021/03/08/99-spam
---

# 99% spam — wingolog

## [99% spam](/archives/2021/03/08/99-spam)

8 March 2021 2:36 PM

- [spam](/tags/spam)
- [bayes](/tags/bayes)
- [classifier](/tags/classifier)
- [oh god](/tags/oh%20god)
- [more spam](/tags/more%20spam)
- [comments](/tags/comments)
- [meta](/tags/meta)

Hey all, happy new year apparently! A quick service update on the old wingolog. For some time the site has been drowning in spam comments, despite [my best efforts to point a bayesian classifier at the problem](https://wingolog.org/archives/2017/03/06/its-probably-spam).

I don't keep logs of the number of attempts at posting comments that don't pass the classifier. But what I can say is that since I put in the classifier around 4 years ago, about 2500 comments a year made it through -- enough to turn the comment section into a bit of a dump. Icky, right??

At the same time of course, that's too many comments to triage manually, so I never got around to fixing the problem. So in fact I had two problems: lots 'o spam, and lots 'o incoming spam.

With regards to the existing spam, I took a heavyhanded approach. I took a look at all good emails and URLs that people had submitted for comments prior to 2017, assuming they were triaged. Then I made a goodlist of comments since 2017 that had those comments or emails. There were very few of those -- maybe 50 or 70 or so.

Then I took a look at when comments were made relative to the posts. Turns out, 98.3% of comments were made more than 21 days after their parent post was published -- and sometimes years afterwards. I used that as a first filter, that if a post wasn't from a known poster, and was made 3 weeks or more after the post, I just classified it as spam.

The list of comments made within 3 weeks of the parent post was small enough for me to triage manually, and there I was able to save a bit of wheat from the chaff. In the end, though, the result of winnowing was less than 1% of what went in.

As I don't really want to babysit this wobsite, I'll keep this policy in place in the future -- comments will be open for a while after articles are posted. Hopefully that should keep the ol' wingolog in tidy shape going forward, while still permitting people to comment -- something I have really appreciated in the past.

So happy 2021 to everybody, may the vaccine gods shine upon your shoulders, and happy hacking :)

## related articles

- [service update](/archives/2023/12/14/service-update)
- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [a wingolog user's manual](/archives/2014/08/27/a-wingolog-users-manual)
- [nombrilliant, actually](/archives/2024/01/18/nombrilliant-actually)
- [sir talks-a-lot](/archives/2023/12/12/sir-talks-a-lot)
- [colophonwards](/archives/2023/12/05/colophonwards)

### 6 responses

1. [Arne Babenhauserheide](https://www.draketo.de/index) says:[8 March 2021 3:00 PM](#e1caa63dbd2ce99b49bfa5c53cae8670cadc75c2)

   Thank you!

2. Dale says:[8 March 2021 5:54 PM](#51e4b36664a3b4cd9562a11971260fd1fe77ede2)

   Yey!

3. Delgarde says:[9 March 2021 6:52 AM](#9325d209f7287659a52a93c2935235f8bdd849d0)

   Sounds about right... most people reading a blog post will do so within a few days, since they're following via their news feeds. And so I rarely see much value in commenting after that period, since I know that hardly anybody will ever read my comment.

4. tomás zerolo says:[12 March 2021 10:10 AM](#6118af4ce4a5603e60ed48955d22686c12f213aa)

   You could provide a button "report as spam" and feed that to your Bayesian thingy, weighted with some confidence factor.

   Then adjust those factors via backprop... OK, OK, I'll shut up :)

   Anyway, nice to read something here. I don't feel that well over at twitterland.

5. [Alaric Snell-Pym](https://www.snell-pym.org.uk/alaric/) says:[15 March 2021 10:16 AM](#2f6871c74317e167999853ca47ee3c57d10a5f63)

   The most important feature of any anti-spam system is that it's low enough effort to you that you don't just turn comments off; the cost of it must be sufficiently less than the value to you of allowing comments!

6. Brandon says:[26 March 2021 6:33 AM](#1380bf58f06df157ce4770a4c79ea7189deba8eb)

   Let me just say that I do enjoy reading your posts, and I'm sorry the spam has been such a problem. Quite why any spammers bother with your little corner of the tubes is beyond me. They're just outing themselves.

   But frankly, it's insulting when your spam detector flags my actual hand-written post as spam. I think you need to delegate this problem to a piece of software or service that is battle-hardened. The web isn't what it was 10 or 15 years ago. You can't just serve this stuff from an obscure domain and hope for the best.

Comments are closed.
