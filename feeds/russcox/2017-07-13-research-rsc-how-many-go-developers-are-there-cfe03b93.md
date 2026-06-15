---
title: 'research!rsc: How Many Go Developers Are There?'
url: https://research.swtch.com/gophercount
published: "2017-07-13T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/gophercount
---

# research!rsc: How Many Go Developers Are There?

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# How Many Go Developers Are There? Russ Cox July 13, 2017 Updated August 5, 2021. *research.swtch.com/gophercount* Posted on Thursday, July 13, 2017. Updated Thursday, August 5, 2021.

How many Go developers are there in the world? *Around two million.*

As of August 2021, my best estimate is between 1.6 and 2.5 million.

Previously:

In July 2017, I estimated between half a million and a million.

In July 2018, I estimated between 0.8 and 1.6 million.

In November 2019, I estimated between 1.15 and 1.96 million.

## [Approach](\#approach)

My approach is to compute:

*Number of Go Developers* =    *Number of Software Developers*   ×   *Fraction using Go*

Then we need to answer how many software developers there are in the world and what percentage of them are using Go.

## [Number of Software Developers](\#number_of_software_developers)

How many software developers are there in the world?

In January 2014, [InfoQ reported](https://www.infoq.com/news/2014/01/IDC-software-developers) that IDC published a report (no longer available online, it would seem) estimating that there were 11,005,000 “professional software developers” and 7,534,500 “hobbyist software developers,” giving a total estimate of 18,539,500.

In October 2016, Evans Data Corporation issued a [press release](https://evansdata.com/press/viewRelease.php?pressID=244) advertising their “ [Global Developer Population and Demographic Study 2016](https://web.archive.org/web/20160730050104/http://www.evansdata.com/reports/viewRelease.php?reportID=9)” in which they estimated the total worldwide population of software developers to be 21 million.

Maybe the Evans estimate is too high. The details of their methodology are key to their business and therefore not revealed publicly, so we can’t easily tell how strict or loose their definition of developer is. In January 2017, PK of the DQYDJ blog posted an analysis titled “ [How Many Developers are There in America, and Where Do They Live?](https://dqydj.com/number-of-developers-in-america-and-per-state/),” That post, which includes an admirably detailed methodology section, used data from the 2016 American Census Survey (ACS) and included these employment categories as “strict” software developers:

- Computer Scientists and Systems Analysts / Network Systems Analysts / Web Developers

- Computer Programmers

- Software Developers, Applications and Systems Software

- Database Administrators

Using that list, PK arrived at a total of 3,357,626 software developers in the United States. The post then added two less strict categories, which expanded the total to 4,185,114 software developers. A conservative estimate of the number of software developers worldwide would probably include only PK’s “strict” category, about 80% of the United States total. If we assume conservatively that the Evans estimate was similarly loose and that the 80% ratio holds worldwide, we can make the Evans estimate stricter by multiplying it by the same 80%, arriving at 16.8 million.

Maybe the Evans estimate is too low. In May 2017, RedMonk blogger [James Governor reported](http://redmonk.com/jgovernor/2017/05/26/just-how-many-darned-developers-are-there-in-the-world-github-is-puzzled/) that, in a recent speech, GitHub CEO Chris Wanstrath claimed GitHub has 21 million active users and GitHub’s Atom editor has 2 million active users (who haven’t turned off metrics and tracking) and concluded that the IDC and Evans estimates are therefore too low. Governor went on to give a “wild assed guess” of 35 million developers worldwide.

*Based on all this (and ignoring wild-assed guesses), my estimate in July 2017 was that the number of software developers worldwide was likely to be in the range 16.8–21 million.*

The [2018 Evans Data Global Developer Population and Demographic Study](https://evansdata.com/press/viewRelease.php?pressID=268) estimated 23 million developers worldwide, up from 21 million in the 2016 survey.

*Based on the confidence range in 2017 being 16.8–21 million, my estimate in July 2018 applied the Evans Data 10% growth to arrive at 18.4–23 million developers in 2018.*

The [2019 Evans Data Global Development Survey](https://evansdata.com/press/viewRelease.php?pressID=278) estimated 23.9 million developers worldwide in May 2019, up from 23 million in the 2018 survey.

The [IDC Worldwide Developer Census, 2018](https://www.idc.com/getdoc.jsp?containerId=US44363318) estimated 22.30 million developers worldwide, up from [21.25 million in 2017](https://www.idc.com/getdoc.jsp?containerId=US44389218).

SlashData’s [Global Developer Population 2019](https://slashdata-website-cms.s3.amazonaws.com/sample_reports/EiWEyM5bfZe1Kug_.pdf) estimates 14.7 million developers in Q2 2017, 15.7 million in Q4 2017, 16.9 million in Q2 2018, and 18.9 million in Q4 2018, extrapolating to “at least 21M developers by the end of 2019 and possibly upwards of 23M.” Oddly, SlashData’s [Developer Economics State of the Developer Nation 17th Edition](https://www.slashdata.co/free-resources/state-of-the-developer-nation-17th-edition) estimated “18 million active software developers in the world” as of Q2 2019. It is unclear if this was rounded down from the 18.9, if SlashData believes the number of developers decreased in the first half of 2019, or if the numbers are completely unrelated.

*Based on these, my estimate in November 2019 is that the number of developers worldwide is likely to be in the range 18.9–23.9 million developers.*

The Evans Data Corporation Worldwide Developer Population and Demographic Study 2020, Volume 2 estimated 24.5 million developers worldwide, with lower growth than predicted attributed to the pandemic.

The [IDC Worldwide Developer Census, 2020](https://www.idc.com/getdoc.jsp?containerId=US47513521) estimated 26.2 million developers worldwide in January 2020.

SlashData’s [Developer Economics State of the Developer Nation 20th Edition](https://www.slashdata.co/free-resources/developer-economics-state-of-the-developer-nation-20th-edition) estimated 24.3 million developers worldwide.

These estimates seem to be converging, but to be conservative, I’m only going to move the low end halfway from the previous 18.9 to the 24.3 here, producing 21.6 million.

*Based on these, my estimate in August 2021 is that the number of developers worldwide is likely to be in the range 21.6–26.2 million developers.*

## [Fraction using Go](\#fraction_using_go)

What fraction of software developers use Go?

Stack Overflow has been running an annual developer survey for the past few years. In their [2017 survey](https://insights.stackoverflow.com/survey/2017#technology), 4.2% of all respondents and 4.6% of professional developers reported using Go. Unfortunately, we cannot sanity check Go against the year before, because the [2016 survey report](https://insights.stackoverflow.com/survey/2016#technology) cut off the list of popular technologies after Objective-C (6.5%).

O’Reilly has been running annual software developer salary surveys for the past few years as well, and their survey asks about language use. The [2016 worldwide survey](https://www.oreilly.com/ideas/2016-software-development-salary-survey-report#id-kAiqtmuE) reports that Go is used by 3.9% of respondents, while the [2016 European survey](https://www.oreilly.com/ideas/2016-european-software-development-salary-survey#id-4xiji1CA) reports Go is used by 3.3% of respondents. (I derived both these numbers by measuring the bars in the graphs.) The [2017 worldwide survey](https://www.oreilly.com/ideas/2017-software-development-salary-survey#tools) reports in commentary that 4.5% of respondents say they use Go.

Maybe the 4.2–4.6% estimate is too high. Both of these are online surveys with a danger of self-selection bias among respondents, and that developers who don’t answer online surveys use Go much less than those who do. For example, perhaps the surveys are skewed by geography or experience in a way that affects the result. Suppose, I think quite pessimistically, that the survey respondents are only representative of half of the software developer population, and that in the other half, Go is only half as popular. Then if a survey found 4.2% of developers use Go, the real percentage would be three quarters as much, or 3.15%.

*Based on all this, my estimate in July 2017 was that the fraction of software developers using Go worldwide is at least 3% and possibly as high as 4.6%.*

The [2018 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2018/#most-popular-technologies) reports that 7.1% of all developers and 7.2% of professional developers use Go. Applying the same pessimism as last year (multiplying by three quarters) suggests a lower bound of 5.3%, but I’ll be even more conservative and use last year’s 4.6% as a lower bound. (The O’Reilly surveys seem to have stopped being run.)

*Based on this, my estimate in July 2018 was that 4.6–7.1% of developers use Go.*

The [2019 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2019/#most-popular-technologies) reports that 8.2% of all developers and 8.8% of professional developers use Go (compared with 7.1% and 7.2% in 2017).

The [HackerRank 2019 Developer Skills Report](https://research.hackerrank.com/developer-skills/2019#skills2) reports that 8.8% of respondents know Go at the end of 2018 (compared with 6.08% at the end of 2017).

SlashData’s [Developer Economics State of the Developer Nation 17th Edition](https://www.slashdata.co/free-resources/state-of-the-developer-nation-17th-edition) estimated 1.1 million active Go software developers, out of 18 million worldwide, or 6.15%. (That happens to match my usual “three quarters of Stack Overflow” conservative lower bound exactly.)

The [JetBrains Developer Ecosystem Survey 2019](https://www.jetbrains.com/lp/devecosystem-2019/) reports that 18% of developers used Go in 2018. That number seems too high, so I will discount it for now.

Evans Data Corporation’s Global Development Survey reports even higher (and harder to believe) percentages for Go usage in the non-public part of the survey.

*Based on all this, my estimate in November 2019 is that 6.1–8.2% of developers use Go.*

The [2020 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2020#technology-programming-scripting-and-markup-languages-all-respondents) reports that 8.8% of all developers and 9.4% of professional developers use Go.

The [2021 Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey/2020#technology-programming-scripting-and-markup-languages-all-respondents) reports that 9.55% of all developers and 10.51% of professional developers use Go.

SlashData’s [Developer Economics State of the Developer Nation 20th Edition](https://www.slashdata.co/free-resources/developer-economics-state-of-the-developer-nation-20th-edition) estimated 2.1 million active Go software developers, out of 24.3 million worldwide, or 8.6%.

The [JetBrains Developer Ecosystem Survey 2021](https://www.jetbrains.com/lp/devecosystem-2021/#Main_programming-languages) reports that 17% of developers used Go in 2020. That number still seems too high, so I will continue to discount it.

Evans Data Corporation’s Global Development Survey 2021 continues to report an even higher (and harder to believe) percentage for Go usage: 22.2% of developers spending at least 10% of their programming time using Go, roughly on par with Kotlin.

Like in the total developer population, the believable numbers seem to be converging, but to be conservative, I’m only going to move the low end halfway from the previous 6.1 to the current 8.6, producing 7.4%.

*Based on all this, my estimate in August 2021 is that 7.4–9.5% of developers use Go.*

## [Number of Go Developers](\#number_of_go_developers)

How many Go developers are there? Multiply the low developer count and Go percentages and the high ones.

In July 2017, 3–4.6% of 16.8–21 million yielded an estimate of 0.50–0.97 million Go developers.

In July 2018, 4.6–7.1% of 18.4–23 million yielded an estimate of 0.85–1.63 million Go developers.

In November 2019, 6.1–8.2% of 18.9–23.9 million yields an estimate of 1.15–1.96 million Go developers.

In August 2021, 7.4–9.5% of 21.6–26.2 million yields an estimate of 1.6–2.5 million Go developers.
