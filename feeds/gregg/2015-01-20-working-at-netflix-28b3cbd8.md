---
title: Working at Netflix
url: https://www.brendangregg.com/blog/2015-01-20/working-at-netflix.html
published: "2015-01-20T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-01-20/working-at-netflix.html
---

# Working at Netflix

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

# [Brendan Gregg's Blog](/blog/index.html)

[home](/blog/index.html)

## Working at Netflix

20 Jan 2015

I've been at Netflix now for several months, and have found it to be an amazing place to work. What has surprised me most is the culture, how different it is to other companies, and how well it works.

In this post I'll describe my experience at Netflix: starting with recruiting, the culture, my work as a performance architect, and finally our mission. I'm excited by what we are doing as a company, and I hope this can (or can continue to) inspire positive changes across our industry.

I hope to also address the questions I'm frequently asked nowadays, including: “Is the culture deck true?", and “How is performance work at Netflix?".

No one at Netflix asked me to write this, and these are my own opinions.

## Recruiting

My first direct experience with Netflix was the hiring process. Netflix recruiting is outstanding. I was scheduled quickly and with the right people: my would-be manager, co-workers, and higher management. The focus was on finding out if we were a good fit for each other.

I had had quite different experiences interviewing with some other companies. One major tech company was not just initially slow to interview, but focused the interviews on topics unrelated to my expected role. I was told that the hiring process had known issues, which couldn't be fixed. Really? If hiring is broken, then what else is broken? And who would my co-workers be, who passed an unrelated interview? This was my first exposure to the company, and it showed a culture of "stupid things happen, we know they’re stupid, and they can't be fixed". (I don't even think the recruiters were to blame; this was a company problem.)

The question of compensation was also handled differently at Netflix. In many hiring negotiations, both parties try to bluff their way around this topic. With Netflix, I was encouraged to find out my market worth and discuss it with them, so that we could both agree on what constituted a good salary. They also did their own research by collecting data from candidates and other companies throughout the hiring process, to ensure the offer was top of market. I think some other companies are terrified of staff discovering what they are really worth! But this openness and honesty was characteristic of the Netflix recruiting process.

As part of the recruiting process, I was encouraged to study the [culture deck](http://www.slideshare.net/reed2001/culture-1798664). I did, and found it attractive.

## Culture

Many companies talk about about how great their culture is, but this is more aspirational than reality. After joining the company, the real culture is learned by word of mouth, or trial and error. However, with Netflix, the culture deck is true. I think its emphasis in the recruitment stage reinforces this, as everyone learns how to act before they walk in the door.

One of the key principles in the culture is "freedom and responsibility": you have the freedom to do the right thing – provided you take responsibility. Management’s role is to provide context and help, not be an obstacle. This is awesome: I have a long list of projects I want to do that will help Netflix, and I'm in an environment that helps me do it.

But how does this work? It's not freedom gone wild. You could introduce a new technology, for example, provided you plan who will service and maintain it, how it's debugged when it fails, how fault tolerance works, etc. Freedom and responsibility.

This works because Netflix hires professionals: people who have good judgement to start with and who use freedom wisely. People with self-discipline. This doesn't mean we're always right: freedom and responsibility includes having the courage and curiosity to try risky projects, when it makes sense to do so, as these can lead to important innovations. The focus is always on making a positive impact for the company.

Apart from professionalism, Netflix also aims to hire high performers. People who are self-driven, highly productive, and who also work well with others. This means no "brilliant jerks", who are explicitly not welcome (this has been described as a polite version of the "no asshole rule" \[ [1](http://bobsutton.typepad.com/my_weblog/2009/09/netflix-culture-an-amazing-slideshow.html)\]).

There's much more in the culture deck. I may be able to help explain it by describing what I think Netflix is **not**:

- It's not the kind of company where known stupid things happen that can't be fixed (like the flawed recruiting process, or the antics parodied by Dilbert).
- It's not the kind of company where managers or departments hate each other, and use their roles to conduct political warfare.
- It's not the kind of company where bad mistakes are frequently made, and no one is held accountable.
- It's not the kind of company where all effort is on fire-fighting, and little on fire-proofing.
- It's not the kind of company that locks its engineers in the basement, for fear of them being poached.
- It's not a company where an unhealthy work/life balance is either encouraged or necessary.
- It's not a company suffering Not Invented Here syndrome, No Bad News syndrome, or groupthink.
- And it's not a company where waste or insanity run rampant.

It's like... a company run by adults instead of children! (Adultlike behavior has even been described previously as a tenet of hiring: "hire, reward, and tolerate only fully formed adults" \[ [2](https://hbr.org/2014/01/how-netflix-reinvented-hr/ar/1)\].)

While the Netflix culture attracted me, it doesn't suit everyone, as described on [slide 38](http://www.slideshare.net/reed2001/culture-1798664/38). And that's ok. Part of being open (and honest) about our culture is that it helps people self-select.

## Performance Engineering

There are a few companies doing important work in performance, and Netflix is one of them. With over 50 million subscribers, the largest cloud environment, and handling over a third of the US Internet traffic at night, there are numerous opportunities for performance engineering. This involves not just applying existing practices, but developing the state of the art.

I'm working on many technologies: AWS, Linux, FreeBSD, Java, Node.js, Perl, Python, Cassandra, Nginx, ftrace, perf\_events, and eBPF to name a few. There are many technologies we've developed ourselves, which are typically open sourced, including the rxNetty reactive framework, Atlas for performance monitoring, and (coming up) Vector for instance analysis. There's also hardware performance work (for the FreeBSD appliances) and capacity planning.

My day-to-day work includes resolving immediate issues of poor performance, and short- and long-term projects. We're using Linux on our cloud, where advanced performance analysis tools have historically been lacking. I have the freedom to figure out what best to do, which has included developing ftrace and perf\_events performance analysis tools, for short-term wins ( [perf-tools](https://github.com/brendangregg/perf-tools)); Java hotspot hacking, for short- and long-term wins; and eBPF testing, for long-term wins. I'll be putting some time into the other tracers (SystemTap, ktap, LTTng), to find wins from them, too.

The FreeBSD Open Connect Appliances, our CDN which streams the actual content, are amazing to work on. FreeBSD provides the most advanced performance analysis environment, including many standard tools as well as pmcstat and DTrace (I summarized these in my MeetBSDCA 2014 talk: [slides](http://www.slideshare.net/brendangregg/meetbsd2014-performance-analysis), [video](https://www.youtube.com/watch?v=uvKMptfXtdo)). I was delighted to recently develop [CPI flame graphs](http://www.brendangregg.com/blog/2014-10-31/cpi-flame-graphs.html) on the OCAs, using pmcstat.

Other specific work I've been posting here on my blog. In particular, [From Clouds to Roots](/blog/2014-09-27/from-clouds-to-roots.html) shows the full performance analysis process for our cloud, from cloud-wide to instance-level tools. I've found analysis of Linux instances to be easier than I’d feared, thanks to ftrace and perf\_events being built into the kernel. The biggest challenge has been determining low-level CPU behavior in the cloud, without current access to CPU performance counters. However, I've made some progress using [MSRs instead](http://www.brendangregg.com/blog/2014-09-15/the-msrs-of-ec2.html).

I'm part of a great team, Performance and Reliability Engineering. I'm not just applying my skills, but learning more from other colleagues, and developing those skills in our environment. We're also expanding our team, and hiring [performance](http://jobs.netflix.com/jobs.php?id=NFX01535) and [systems](http://jobs.netflix.com/jobs.php?id=NFX01887) engineering roles (if interested, contact [Coburn](https://twitter.com/coburnw), our manager).

## Mission

Netflix’s mission is to change how entertainment is consumed worldwide, by building a good product that people choose to buy. It's exciting: we are pioneering the modern age of entertainment, and taking on all the technical and political challenges that this involves.

We aren't building a technical facade, where the real mission is to sell the company. We aren't winning by unsavory sales, legal, or marketing tactics. And our customers are not unwittingly themselves the product (we don't read their private emails). We want to win by making a product so good that people choose to buy it. We're an honest company.

## Conclusion

When I joined Netflix, I expected many aspects to be excellent: the technical challenges, my colleagues, the compensation, the mission, and the ambition of the company – and they are. What surprised me most by its excellence was the culture. It's made me think differently about our industry: Netflix is proving that company culture doesn't just have to be accepted: it can be engineered to be positive.

In this post I described my opinions of Netflix after ten months at the company. I'm motivated to write because I think the topic of positive company cultures is worth discussing. For more about the Netflix culture, apart from the culture deck, you can read [how Netflix reinvented HR](https://hbr.org/2014/01/how-netflix-reinvented-hr/ar/1), [behind the slides](http://blog.slideshare.net/2014/06/11/inside-the-netflix-culture-deck/), and [the woman behind the Netflix culture doc](http://firstround.com/article/The-woman-behind-the-Netflix-Culture-doc). Maybe you can experience this culture directly (we are hiring), or perhaps more of our industry can adopt or engineer their own positive cultures.

UPDATE: [Working at Netflix 2016](http://www.brendangregg.com/blog/2016-03-30/working-at-netflix-2016.html).

UPDATE: [Working at Netflix 2017](http://www.brendangregg.com/blog/2017-05-16/working-at-netflix-2017.html).

*Update: Many people have been emailing me their resumes. I'm glad they are interested in working at Netflix, but I'm an engineer, not a hiring manager. Please use jobs.netflix.com, which will send your details to the right people.*

---

Click here for Disqus comments (ad supported).

*You are welcome to comment here, but I've been meaning to switch comment systems one day and I don't know yet if I can preserve existing comments (I'll try to find a way).*

[comments powered by Disqus](http://disqus.com)

---

Site Navigation

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

---

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

Copyright 2025 Brendan Gregg.

[About this blog](/blog/about.html)
