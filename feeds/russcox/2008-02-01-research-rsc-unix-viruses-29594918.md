---
title: 'research!rsc: Unix Viruses'
url: https://research.swtch.com/virus
published: "2008-02-01T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/virus
---

# research!rsc: Unix Viruses

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Unix Viruses Russ Cox February 1, 2008 *research.swtch.com/virus* Posted on Friday, February 1, 2008.

Disk-based computer viruses came of age with MS-DOS; network-based viruses came of age with Microsoft Windows. (The most stunning recent example is the widespread [Storm worm](http://www.schneier.com/blog/archives/2007/10/the_storm_worm.html) which is at this very minute probably sending you pump-and-dump and greeting card spam.)

Most Linux and OS X users have a frustratingly smug attitude of “we're immune to viruses, because we use Unix.” This is obviously false: the [original Internet worm](http://query.nytimes.com/gst/fullpage.html?res=940DE2D81038F937A35752C1A96E948260&pagewanted=all) ran on Unix. The simple security mechanisms in Unix-based systems do raise the virus-writing bar slightly, but a more likely explanation of the lack of viruses on Unix-based systems is their lack of current market share: Windows users are simply a much bigger and therefore more attractive target.

Especially for these smug Unix users, it should be sobering to read Tom Duff's 1987 technical report “ [**Viral Attacks on UNIX® System Security**](https://9p.io/who/dmr/tdvirus.pdf),” which describes a simple, benign disk-based virus targeted at Unix executables. Even working under the constraints of the Unix permission bits, Duff managed to infect 466 files across 46 systems over a period of a few months, including an experimental “secure” version of Unix (to its credit, the kernel on that system did detect the virus). Duff's paper also describes a simple virus written in shell script. He cautions:

> However sorely you are tempted, *do not* run this code. It got loose on my machine while being debugged for inclusion in this paper, and within an hour had infected about 140 files, at which time several copies were energetically seeking other files to infect, running the machine's load average, normally between .05 and 1.25, up to about 17. I had to stop the machine in the middle of a work day and spend three hours scouring the disks, earning the ire of ten or so co-workers. I feel extremely fortunate that it did not escape onto the Datakit network.

Doug McIlroy's 1989 paper “ [Virology 101](http://www.cs.dartmouth.edu/%7Edoug/v101.ps.gz)” (gzipped PostScript) carries the shell script virus farther, developing progressively more virulent strains. McIlroy summarizes his paper:

> There is nothing mysterious about computer viruses. A working, but easily observable, virus can be written in a few lines of code. Although particular virus attacks may be guarded against, no general defense within one domain of reference is possible; viruses are a natural consequence of stored-program computation. Like other hazards of technology, their thread may be mitigated by cautious behavior and community sanctions.

(Finally, of course, there is Thompson's “ [Reflections on Trusting Trust](http://cm.bell-labs.com/who/ken/trust.html),” but that's a discussion for another day.)

(Comments originally posted via Blogger.)

- [Maht](http://www.blogger.com/profile/01863908675256558774)(February 2, 2008 1:59 PM)This post has been removed by the author.

- [Maht](http://www.blogger.com/profile/01863908675256558774)(February 2, 2008 2:00 PM) Hi Russ, I'm enjoying your series.

After reading Doug's paper I was curious to read :

J. A. Reeds, /bin/sh: The biggest UNIX“ security Loophole,11217-840302-04TM, AT&T Bell

Laboratories, Murray Hill, NJ (1984).

but a web search turns up nothing,

So I'm mentioning it here in case someone knows how I can find a copy, thanks.

- [Renaissance](http://www.blogger.com/profile/10866530772307754358)(February 4, 2008 8:03 PM) Fred Cohen invented the first computer virus which ran on VAX VMS in the 1993-1984 time frame at USC. My memory is vague but it was probably 1983. He had his office across the hall from the lab where I worked on 6th floor Powell Hall. Since we were neighbors and fellow hackers when he got it working we were the first people he showed it to. He published about it soon afterward so it would not surprise me if a Unix virus was created shortly afterward.

- [Sorin T](http://www.blogger.com/profile/10903971441214777545)(February 25, 2008 9:27 AM) If I remember well Fred Cohen is saying in his book also 1983 .. he also mentions about trying to talk with IBM/MS \[and others\] about MSDOS design which ignored totally such things \[viruses\] but with no success. And he always tried to show the problems in OS/net service design/architecture and not exploiting a particular bug ..

- Anonymous (July 1, 2011 2:42 PM) You can't mention Reflections on Trusting Trust without also mentioning David Wheeler's Countering Trusting Trust - http://www.dwheeler.com/trusting-trust/ . It's practically obligatory to include it as a rider :)
