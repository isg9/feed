---
title: poverty, riches — wingolog
url: https://wingolog.org/archives/2009/05/14/poverty-riches
published: "2009-05-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/05/14/poverty-riches
---

# poverty, riches — wingolog

## [poverty, riches](/archives/2009/05/14/poverty-riches)

14 May 2009 7:31 PM

- [poverty](/tags/poverty)
- [namibia](/tags/namibia)
- [riches](/tags/riches)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [dybvig](/tags/dybvig)
- [inlining](/tags/inlining)
- [optimization](/tags/optimization)

**riches, poverty**

[![](//wingolog.org/photos//2004/12/8/imag0053.normal.jpg)](//wingolog.org/photos/index.py/photos/207)

How many times have you heard the phrase, "the poorest of the poor, those that live on less than a dollar a day"?

I am tired of it.

You can grow your own food, and they call you "poor". Or you can buy transgenic, pesticide-laced, flavorless tomatoes at the store, and they call you "rich".

Poverty and riches do exist, but they have nothing to do with dollars.

**hack reading**

[*Fast and Effective Procedure Inlining*](http://www.cs.indiana.edu/~dyb/pubs/inlining.pdf), by Oscar Waddell and Kent Dybvig.

Great, great paper. It could be my ignorance, but I never realized that copy propagation, dead code elimination, constant folding, and procedure inlining were all the same operation.

I think I'm going to implement their algorithm soon. It should be straightforward, given that the new high-level intermediate language that I'm working on for Guile is basically the same as their language.

## related articles

- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [partial evaluation in guile](/archives/2011/10/11/partial-evaluation-in-guile)
- [micro macro story time](/archives/2024/01/11/micro-macro-story-time)
- [cross-module inlining in guile](/archives/2021/05/13/cross-module-inlining-in-guile)
- [fibs, lies, and benchmarks](/archives/2019/06/26/fibs-lies-and-benchmarks)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### 23 responses

1. anonymous says:[14 May 2009 8:19 PM](#def42d8bb2766ee8c3ef7fc242f0c24886ceaeb6)

   Where are you from? Probrably a first world country.

   You obviously don't know what poverty is.

2. [John (J5) Palmieri](http://j5live.com) says:[14 May 2009 8:26 PM](#f6ab374dedcc292e9755237815473aa8d530837e)

   Money is used as a way to quantify rich/poor relative to an average. Of course dollar a day is a nice round number so it can be said to be an oversimplification used to market attention to these issues. If a person can subside on food they grow then that would factor into the "dollar a day" measurement. Unfortunately when we are talking about "dollar a day" it usually refers not to poverty but to "extreme poverty". Those in landlocked, arid countries which have little or no fertile ground to subsist on and no effective transportation routes to transport goods they do make, mine, etc. Extreme poverty is highlighted by those who have fallen into the poverty trap with no way of digging out on their own. Also, those who subsist can fall into the trap if one year the crops fail or disease hits. Even those who farm in abundance can fall into the trap if they have no effective means of trading their excess for necessities or to buttress against leaner years. For better or worse, since money can be traded for a plethora of goods and services that is the stick by which we measure.

   If you are interested, a good read on the subject is Jeffrey Sachs the End of Poverty.

3. [wingo](http://wingolog.org/) says:[14 May 2009 8:28 PM](#2ce09cc9e68892f3f856a0c6eb74e94f1239bced)

   Hi anon, and welcome.

   I am American. I live in Spain. I lived in Namibia, in southern Africa, for two years, some years ago. I know poverty, and I know riches.

   You sound like you know poverty too. You can have no money and be poor. You can also have no money and be rich. All I'm saying is that wealth and number of dollars are not the same.

4. Rob J. Caskey of Athens, GA says:[14 May 2009 8:29 PM](#1c7a36c385643ea05516ec6903f4db811568f11a)

   Wingo,

   So you would propose that we reckon the rich not by their bank-accounts but by their tomatoes?

   --Rob

5. ac says:[14 May 2009 8:36 PM](#699432c2a6b34ff973d988a4e3144fb0edda80b2)

   Ignant. There is a difference between "being able to" and "having to" grow your own food.

6. anonymous says:[14 May 2009 8:43 PM](#dc3e125ba5177797da49ce29d0fa5cc8c4581472)

   Although you can say that in a philosophic way, the practical way is way other.

   I am from a third world country. Always been. See, if you don't have money, you are screwed, That's it. We are a very happy country and we always see the "bright sight of the life". But it is definitely a illusion to think that you "can grow your on food" and get out of it. One have health, education, bad luck, etc. Even if you think about natives - which, by the way, every country likes to emphasize when they talk about us - they do still need government help. There are floods, diseases, food shortages.

   One could think at a different kind of economy and society, and that would be great. But, at ours, if you say "those who live with one dollar", you can bet your pants that they are poor, and more, that they need help, from others, from the government or whomever.

7. [Maciej Piechotka](http://blog.pichotka.com.pl) says:[14 May 2009 8:43 PM](#9417bb4e1b3a67ce485d5b8c3dfc390b441a5fb4)

   And yes and no. Living for $1 in the so-called 3rd world\[1\] is different then living for $1 in Poland (not mentioning richer coutries). First of all $1 have much more purchesing power then in richer countries. For $1 I could buy may be a sandwich - but it'll not be sufficient to find anything to drink. Defiitly one would not live long. However they do as:

   1\. From various resons some goods are avaible cheaper for them (in dollars - not in work-hours etc.) - the purching power varies from place to place (I live in capital city and I find many goods cheaper on the countryside. It's true that the salary is on average highier - but the spendings also)

   2\. Many goods are not counted into income. Self-grown food is not consider income. AFAIK it is not high-quality food but still. Energy from fire is not income etc.

   Still those who live on $1 are very poor but the scale is non-linear.

   \> You obviously don't know what poverty is.

   Given that unlikely those people have internet access - than yes. Although I have to disagree with statement >>You can grow your own food, and they call you "poor". Or you can buy transgenic, pesticide-laced, flavorless tomatoes at the store, and they call you "rich".<< but I'd like to point out that the value of income is non-linear.

   \[1\] Name it developing world or anything els if you find it more politically correct.

   PS. First you ask about 'favorite number'. I gave e and it rejects it ;) on ground it cannot conver it. Please be more specific...

8. [wingo](http://wingolog.org/) says:[14 May 2009 8:44 PM](#4ff3221277d431007fdd63377df3ca4d8aa45dd1)

   @Rob: Yes, in metric metaphorical tomatoes!

   Your question of measurement is a good one, however.

9. anonymous says:[14 May 2009 8:48 PM](#27bfa2e1f051640ab5a558c2ad3ebb1b8f46b852)

   \> You obviously don't know what poverty is.

   > Given that unlikely those people have internet access - than yes.

   You too don't know what poverty is.

10. [John (J5) Palmieri](http://j5live.com) says:[14 May 2009 8:49 PM](#834ca153cb346dc5f54ac7d5b4527ab1f9eece2f)

    Wingo,

    I do agree that these stale mechanical measures do suck the humanity out of the situation. I believe it is Bhutan that measures not only their Gross National Product but also their Gross National Happiness. All of their government decisions take into account both measures.

11. [Maciej Piechotka](http://blog.piechotka.com.pl/) says:[14 May 2009 9:09 PM](#715ef291ee9ab3260a5261d72289def6daf3eb17)

    > Even if you think about natives - which, by the way, every country likes to emphasize when they talk about us - they do still need government help. There are floods, diseases, food shortages.

    You'll probably disagree with me (I have very upopular opinion as anarchocapitalist) but I'd say they do not need gov help as keeping them poor is due to various goverment interventions. To begin with if not subsidies in the high and middle income countries the trade would be much highier (those countries have much highier comparative advantages). Also some of the 'social' policies which are tried to be implemented in such countries as thay prevent from capital accumulation (to which wages tends to be proportional), capital inflow\[1\] etc. Thay can also lead to a public debt which finally have to be financed by tax increase - direct or inflationary\[2\].

    \[1\] By capital I do not mean money. Capital in economics are tools, factories etc. Depending on economist education can be also stated as capital (instead of labour).

    \[2\] Actually it eventually leads to the same - increase in the time preferences which leads to bigger consumption instead of savings/investments. As savings/investment are the thing that leads to increase of productivity (only saved grain can be use to plant in the following year, only wood saved from fire can be used to make a hammer etc.).

    I'm aware it is contrary to the Keynisian "Paradox of savings" but I do not find it logical. As Adam Smith wrote "What is logical on familly level is not madness for country" \[this fragment was retlaslated into English as I read Wealth in my native language\]

12. [wingo](http://wingolog.org/) says:[14 May 2009 9:11 PM](#ffac7f156af4d027eb48e39a68570de7319306d1)

    Howdy John. Your statement about Bhutan rings a distant bell, though I don't recall from where.

    Let's take another analogy. Most of my readers work with free software in some regard. We are rich in free software. Yet this richness only corresponds weakly with dollar flow.

    Yet we so want to quantify our richness -- I have run sloccount many times over my programs :) But this is just aping a system that is not ours.

    In the same sense, the agricultural systems that are traditional and in harmony with the ecosystem, on every part of the planet, are /rich/ -- but that richness is only weakly correlated with their own national currencies, not to mention dollars.

    I'm just tired of evaluating things in terms of a broken system. (See also, the gross lies of the recent financial bubble and collapse. Things are /exactly/ the same as they were a year ago, in a material sense; yet we remain prisoners to a financial fiction.)

13. [Maciej Piechotka](http://blog.piechotka.com.pl/) says:[14 May 2009 9:45 PM](#7e9cf28eeb7a64db6d781047779cd33b8e05fc8b)

    > \> You obviously don't know what poverty is.
    >
    > > Given that unlikely those people have internet access - than yes.
    >
    > You too don't know what poverty is.

    Well. Not directly. I had a luck of belonging to the few percent of earth population from high and middle income countries. I visited a few low income countries as a tourist and I have some informations about a few anothers.

    To answer such question I'd say - how would you define 'know'. If you mean 'be' than yes - I do not know and have little chance to know (which as it is lack it neither the reson to be proud of nor the reson to be ashamed of. It is not even a reson of not commenting the ways out as someone who naver had flu can perfectly know how to cure it). If you mean 'seen' - I've seen (but not extream poorness).

    While I can see the reson of commenting the statement "You can grow your own food, and they call you "poor". Or you can buy transgenic, pesticide-laced, flavorless tomatoes at the store, and they call you "rich"." as diet of those people are far from ideal (to be more exact - the tend to terribly undernourish if not at the starving edge) but I cannot see why you pointed my comment - I stated that due to very low division of labour and different relation of prices to USD the $1 in such country is valued much more then in USA. This shift 'allows' them to survive with so low income which would be 'impossible' in highier income.

    And as far as internet is concerned - it was you (probably) who wrote "See, if you don't have money, you are screwed, That's it." - that's basicly it states that most of them (people with income <$1) have problems with access to water not mentioning electricity (not mentioning internet). People with access to the internet in large maiority had always income much highier then $1.

    > In the same sense, the agricultural systems that are traditional and in harmony with the ecosystem, on every part of the planet, are /rich/ -- but that richness is only weakly correlated with their own national currencies, not to mention dollars.

    I'm afraid it is very post-romantic 'big-city' view. Those people biggest current problem is survival. I cannot see also much 'harmony with ecosystem' - protection of enviroment is a 'luxury' about which we care - but for them it may turn out too 'costly'.

14. Ian says:[14 May 2009 9:49 PM](#b88face2d76450e6463b031b9d9923c3c5538f2e)

    "Poverty and riches do exist, but they have nothing to do with dollars."

    I suppose we can all define words however we like. But when we want our definitions to replace common usage, we should do more than make lazy arguments like this one.

15. [wingo](http://wingolog.org/) says:[14 May 2009 9:59 PM](#5035285b72ae7435f556ac4a2111376b4ac08fc7)

    Ian: Here's another thought. In the conventional sense, you can only be rich in things that can be sold. You cannot be rich in things that may only be given.

    The latter category is quite large.

    I am not arguing from a primitive romanticist perspective. I identify with anarchism. The logical outcome of anarchism is a system in which no money is exchanged; yet to me, such a system is richer than the current situation. To me that points to a problem with the conventional definition of richness. Some "primitive" systems are rich in this way.

16. [Maciej Piechotka](http://blog.piechotka.com.pl) says:[14 May 2009 10:21 PM](#17a96a624d20acd16abe23eb4e833f834c493fff)

    > I am not arguing from a primitive romanticist perspective. I identify with anarchism. The logical outcome of anarchism is a system in which no money is exchanged; yet to me, such a system is richer than the current situation. To me that points to a problem with the conventional definition of richness. Some "primitive" systems are rich in this way.

    1\. Well - the different branches of anarchy have different views on the 'results'. I'm for example anarchocapitalist and IMHO the money is market phenomena, where market is defined as volontary transactions (as opposed to current 'markets' of state socialism/state capitalism). Therefore logically money-based exchange will be the most common one in anarchy. However in such free society other ways will also coexist - for example FLOSS. It is more then likely that some people will form a communities in which at least some propety will be shared (everyone will be free to choose his/her lifestyle).

    2\. I've never stated that romaticist perspective is primitive. I guess it may be even consider as more 'civilised' view (preservation of nature etc.). I merly stated that AFAIK we project this view on the farmers which do not necessarly share it as it can be something thay have too less resources to afford.

    3\. They tend to be poor from nearly any definition of poorness I know - as in not having many goods. Which possibly does not make them unhappy.

17. Paul says:[15 May 2009 3:08 AM](#99a6796181896fc22e3aaf6be1f7f28f58a9f75d)

    A dollar is really nothing more then a metric of exchangem. It's only value is the value we agree to give it.

    Growing your own food is all well and good and might be enough to allow you a happy life a lot of the time but what if (or rather when) you get sick or some other circumstance makes things difficult?

    Dollars (or money) makes sense as an expression of poverty because in the inevitable event that you need something you cannot provide for yourself some form of exchange will be necessary.

    The "rich" in your example may buy "transgenic, pesticide-laced, flavorless tomatoes" but ultimately it's due to choice.

    That is where real poverty lays, in the absence of choice.

18. Ludovic Danigo says:[15 May 2009 8:15 AM](#6c61b75bc905da39156399449094c9891677e22e)

    Another sentence I totally dislike is "the third world is getting poorer everyday". It's simply not true, it's much more complex than that.

    For those who don't know them, check out the videos on http://www.gapminder.org/ those statistical studies are very enlightening.

19. [Sharninder](http://geekyninja.com) says:[15 May 2009 9:36 AM](#e94c3c5ece26b4d9965275130166b891793df291)

    You probably are not rich enough to have your own organic farm ;)

    Seriously, being poor is not about the taste of your tomatoes (!!) or where do they come from .. you atleast have an option of growing them any which way you like, they don't ! And no .. most people don't even have the money to buy seeds or grow fancy juicy tomatoes, btw.

    If you haven't lived in poverty, you probably don't understand it.

20. Ian says:[15 May 2009 3:25 PM](#904d007839345a7a25a667ddeae46ef274952705)

    Wingo: Here's another thought. In the conventional sense, you can only be rich in things that can be sold. You cannot be rich in things that may only be given.

    Wingo: The latter category is quite large.

    Yes. But see here, your argument has already softened. Go back to your post's closing sentence and we find, "Poverty and riches do exist, but they have nothing to do with dollars."

    From "nothing to do with dollars" to \[non-dollar things also matter\]. This is a substantial retreat, and totally unacknowledged.

21. keitai says:[16 May 2009 10:01 AM](#b0243585768f3c0bcaf23b61f36f6d05109cfce9)

    from my experience, "dollarless" in kenya don't have "fresh tomatoes" but live on "ugali" and.. nothing but ugali, which is effectively maize porridge/bread hybrid. When there is a celebration, there is chicken stew with ugali.

    i expect the dollarless in other countries as well to live with severe lack of variety in food.

22. pete trachy says:[19 May 2009 0:14 AM](#36d92a833ba0d95a2433c9e27238c7914b691c17)

    lol. I was coming back here to pile on Wingo for his simplistic commentary, but too many have beat me to it.

    The original phrase is not how experts discuss the at need populations of the world, but a simple way of identifying a group of people that is not terribly precise but intended to be media friendly and elicit sympathy. Although it does reflect some underlying biases of society that could be critiqued, the phrase is succinct and probably somewhat effective for its intended use.

    True wealth, as some overarching value of someone's existence, is like intelligence impossible to reify. Dollars unfortunately are the difference between having medical care, good food, and education etc.. And it works as a pretty good shorthand for people whose day to day job is working to improve and save the lives of those without access to those fundamental human rights.

    I'm not sure about these semantic arguments. I appreciated Paul Farmer's statement that he would talk to a person in whatever terms, economic or otherwise, that he needed to in order to communicate with them. Even though he knows he is advocating for basic human rights, he is more concerned with the results at the end of the day than the language used to get there.

23. Jack says:[11 June 2011 2:51 PM](#3ae55b2d27ea94829b6536b5d2f5c57c1fa3007e)

    If you're a subsistence farmer and the rains don't come, your whole family dies unless the WFP get to you first. If your kid gets diarrhoea, they die because you can't afford medicine or even rehydration salts. If you want to sell some of your crop to buy a pan or a hoe or some roofing felt, you don't know whether the buyer has cheated you because your parents couldn't afford to send you to school. If you live on less than a dollar a day, you live in fear of diseases that most westerners think have long been eradicated.

    Poverty has \*everything\* to do with dollars, because what dollars represent is a safety net. If you have no money, you're completely on your own. You don't get access to education or medicine. Your children get respiratory disease because you can't afford to cook on charcoal. Your wife has a 1-in-20 chance of dying during childbirth because your village has no midwife.

    You don't have a clue.

Comments are closed.
