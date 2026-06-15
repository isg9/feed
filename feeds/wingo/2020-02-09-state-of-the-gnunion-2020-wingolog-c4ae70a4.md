---
title: state of the gnunion 2020 — wingolog
url: https://wingolog.org/archives/2020/02/09/state-of-the-gnunion-2020
published: "2020-02-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2020/02/09/state-of-the-gnunion-2020
---

# state of the gnunion 2020 — wingolog

## [state of the gnunion 2020](/archives/2020/02/09/state-of-the-gnunion-2020)

9 February 2020 7:44 PM

- [gnu](/tags/gnu)
- [free software](/tags/free%20software)
- [fsf](/tags/fsf)

Greetings, GNU hackers! This blog post rounds up GNU happenings over 2019. My goal is to celebrate the software we produced over the last year and to help us plan a successful 2020.

Over the past few months I have been discussing project health with a group of GNU maintainers and we were wondering how the project was doing. We had impressions, but little in the way of data. To that end I wrote some scripts to collect dates and versions for all releases made by GNU projects, as far back as data is available.

In 2019, I count **243 releases**, from **98 projects**. Nice! Notably, on `ftp.gnu.org` we have the first stable releases from three projects:

[GNU Guix](https://guix.gnu.org/)GNU Guix is perhaps the most exciting project in GNU these days. It's a package manager! It's a distribution! It's a container construction tool! It's a package-manager-cum-distribution-cum-container-construction-tool! Hearty congratulations to Guix on their [first stable release](https://guix.gnu.org/blog/2019/gnu-guix-1.0.0-released/).[GNU Shepherd](https://gnu.org/s/shepherd/)The GNU Daemon Shepherd is a modern dependency-based `init` service, written in [Guile Scheme](https://gnu.org/s/guile/), and used in Guix. When you install Guix as an operating system, it actually *stages* Scheme programs from the operating system definition into the Shepherd configuration. So cool![GNU Backgammon](https://gnu.org/s/gnubg)Version 1.06.002 is not GNU Backgammon's first stable release, but it is the earliest version which is available on `ftp.gnu.org`. Formerly hosted on the now-defunct `gnubg.org`, GNU Backgammon is a venerable foe, and uses neural networks since before they were cool. Welcome back, GNU Backgammon!

The total release counts above are slightly above what Mike Gerwitz's scripts count in his "GNU Spotlight", posted on the [FSF blog](https://www.fsf.org/blogs/community/gnu-spotlight-with-mike-gerwitz-16-new-gnu-releases-in-january). This could be because in addition to files released on [`ftp.gnu.org`](https://ftp.gnu.org/pub/gnu/), I also manually collected release dates for most packages that upload their software somewhere other than gnu.org. I don't count `alpha.gnu.org` releases, and there were a handful of packages for which I wasn't successful at retrieving their release dates. But as a first approximation, it's a relatively complete data set.

I put my scripts in [git repository](https://gitlab.com/gnumaint/gnumaint/-/blob/master/release-dates/process-dates.scm/) if anyone is interested in playing with the data. Some raw CSV files are there as well.

**where we at?**

Hair toss, check my nails, baby how you GNUing? Hard to tell!

To get us closer to an answer, I calculated the active package count per year. There can be other definitions, but my reading is that an active package is one that has had a stable release within the preceding 3 calendar years. So for 2019, for example, a GNU package is considered active if it had a stable release in 2017, 2018, or 2019. What I got was a graph that looks like this:

![](https://wingolog.org/pub/active-projects-by-year.png)

What we see is nothing before 1991 -- surely pointing to lacunae in my data set -- then a more or less linear rise in active package count until 2002, some stuttering growth rising to a peak in 2014 at 208 active packages, and from there a steady decline down to 153 active packages in 2019.

Of course, as a metric, active package count isn't precisely the same as project health; [GNU ed](https://gnu.org/s/ed/) is indeed the standard editor but it's not GCC. But we need to look for measurements that indirectly indicate project health and this is what I could come up with.

Looking a little deeper, I tabulated the first and last release date for each GNU package, and then grouped them by year. In this graph, the left blue bars indicate the number of packages making their first recorded release, and the right green bars indicate the number of packages making their last release. Obviously a last release in 2019 indicates an active package, so it's to be expected that we have a spike in green bars on the right.

![](https://wingolog.org/pub/terminal-releases-by-year.png)

What this graph indicates is that GNU had an uninterrupted growth phase from its beginning until 2006, with more projects being born than dying. Things are mixed until 2012 or so, and since then we see many more projects making their last release and above all, very few packages "being born".

**where we going?**

I am not sure exactly what steps GNU should take in the future but I hope that this analysis can be a good conversation-starter. I do have some thoughts but will post in a follow-up. Until then, happy hacking in 2020!

## related articles

- [gnu, gnome, and the fsf](/archives/2009/12/13/gnu-gnome-and-the-fsf)
- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [here we go again](/archives/2021/03/25/here-we-go-again)
- [on gnu and on hackers](/archives/2014/08/18/on-gnu-and-on-hackers)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [guix on the framework 13 amd](/archives/2024/02/16/guix-on-the-framework-13-amd)

Comments are closed.
