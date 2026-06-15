---
title: The Real Problem with the US News Rankings
url: https://blog.regehr.org/archives/1558
published: "2017-12-01T02:44:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=1558
---

# The Real Problem with the US News Rankings

The latest list of [Best Global Universities for Computer Science](https://www.usnews.com/education/best-global-universities/computer-science) from US News has not been well received. For example, the Computing Research Association issued [a statement](https://cra.org/cra-statement-us-news-world-report-rankings-computer-science-universities/) saying that “Anyone with knowledge of CS research will see these rankings for what they are – nonsense – and ignore them. But others may be seriously misled.” The CRA statement identifies these problems with the US News rankings:

- They ignore conference publications (many areas of CS publish primarily in conferences).

- US News doesn’t even say which venues are used to compute the publication-based part of the ranking function.

- The reputation-based part of the rankings doesn’t make much sense given the diverse, global nature of the computer science research community.

An additional problem is that it seems to be pretty easy to game this ranking system using money. For example, King Abdulaziz University (Jeddah, Saudi Arabia) has [adopted hiring practices that appear to be designed to do this](https://www.timeshighereducation.com/news/secondary-affiliations-lift-king-abdulaziz-university-in-rankings/2014546.article#survey-answer). Their CS department is ranked #13, compared for example to CMU at #22 and Illinois at #46. I’m trying to avoid being USA-centric and elitist here, but based on some web searches, it is just not possible to objectively rate the [CS department at KAU](http://computing.kau.edu.sa/Default.aspx?Site_ID=611&Lng=EN) as being better than [the one at CMU](https://www.cs.cmu.edu/). US News explains their [ranking methodology here](https://www.usnews.com/education/best-global-universities/articles/subject-rankings-methodology).

To summarize, US News is designed to make money, not to do the CS community any favors. Universities are going to try to maximize their rankings. It’s a pretty banal situation all around.

What I wanted to talk about today is the function of rankings. What are we supposed to do with them? The conclusion I’ve come to is that a closed, opaque ranking such as the one from US News is only good for one thing: codifying and reinforcing a pecking order so that it can be used by people who don’t need or want any more information than a total ordering. This might include, for example, university administrators who would like to know if sending additional resources to a department resulted in a measurable and externally-visible improvement.

The reason everyone’s annoyed with US News is that they’ve upended the established pecking order. But here’s the thing: they could fix this tomorrow and their opaque rankings would still be worthless for people who care about what’s behind the rankings, as opposed to being interested in ranking for its own sake. There has to be a better way.

In contrast, let’s take a look at [CSRankings](http://csrankings.org/), a site that Emery Berger put together using publicly available data from [DBLP](http://dblp.uni-trier.de/). This ranking assigns credit to departments based on the number of top-tier papers published by their full-time faculty, credit for which is split among authors. There’s [a FAQ](http://csrankings.org/faq.html) giving additional details. (There’s a lot of quibbling that could be done about how this all works; I’m not too interested in that.)

The thing that makes this ranking different for practical purposes isn’t the openness of the algorithm and the data set, but rather the way the web site allows us to explore the data. Let’s say that I’m a prospective graduate student interested in operating systems and formal verification. The first thing I can do is select only those areas — now the site shows me the departments that have people who tend to publish heavily in conferences such as SOSP and CAV. Second, I can click on an individual department and see who the key players are in those areas. Third, I can go to these people’s home pages, Google Scholar pages, etc. in order to see what they are specifically doing, and finally I can read their code and papers. I would argue that this is a fundamentally different use of a ranking system: the purpose is to guide me towards details that matter, not to hide everything behind a number.

In summary, I find the complaints about the US News rankings to be a bit off the mark, since even a fixed version of them will provide no insights and no information beyond an opaque ordering. It would just be confirming the status quo instead of refuting it, as their current rankings do. That is what some people want, but it is of little use to faculty and students in the field. A better use of rankings is to serve as a guide for further exploration — for this to happen, the rankings need to be open and connected to more detailed sources of information. CSRankings accomplishes this and it is the tool you should use to explore the productivity of computer science departments. If you don’t like it, you can try to convince Emery to do things differently or else create your own ranking.
