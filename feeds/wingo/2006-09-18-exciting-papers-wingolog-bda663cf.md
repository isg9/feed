---
title: exciting papers! — wingolog
url: https://wingolog.org/archives/2006/09/18/exciting-papers
published: "2006-09-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/09/18/exciting-papers
---

# exciting papers! — wingolog

## [exciting papers!](/archives/2006/09/18/exciting-papers)

18 September 2006 1:08 PM

- [scheme](/tags/scheme)
- [google](/tags/google)
- [distributed systems](/tags/distributed%20systems)
- [r6rs](/tags/r6rs)

I am currently very plussed by three excellent papers.

- [Revised5.91 Report on the Algorithmic Language Scheme](http://www.r6rs.org/r6rs_91.pdf), the latest draft for the long-awaited R6RS.

- [Bigtable: A Distributed Storage System for Structured Data](http://labs.google.com/papers/bigtable.html), describing the database-like store that Google uses as a backend of a lot of their applications.

- [The Chubby Lock Service for Loosely-Coupled Distributed Systems](http://labs.google.com/papers/chubby.html), about a highly available lock service / database / name server.

The first is important, bringing the Scheme up to speed with the implementations, and a bit beyond as well: unicode, a standard compilation-friendly library mechanism (finally!), standard support for `#!`-style scripting, byte vectors, records, standardized exception and condition handling, byte-based i/o, fixnums, quasisyntax (!!!), and more.

The bit about library support is key. Libraries are especially difficult in languages with macros (see [Composable and compilable macros:: you want it when?](http://portal.acm.org/citation.cfm?id=581486&dl=ACM&coll=portal&CFID=11111111&CFTOKEN=2222222)). Yay. Yay!

The other papers are more relevant to my current line of work, building distributed systems for [Fluendo](http://www.fluendo.com/). It's really interesting to see what kinds of reusable pieces Google has managed to factor out of their applications, and the practical experience building tools to make a Google application. From what I can see from these two papers and the previous ones at [their papers page](http://labs.google.com/papers/index.html), they have an extremely strong platform. It is enviable.

I [still feel like Google is a fortress](//wingolog.org/archives/2005/11/01/google-profiler-a-nice-gesture-but). This time in addition to the processional desire, paper releases are pushed by their recruiting department as well -- see the conspicuous "Why work at Google?" link once you have finished digesting their papers.

I've been thinking a bit about how to build a distributed "Web 2.0" app using peer-to-peer technologies. You would need what they have, a mapreduce/sawzall approach to data processing, but instead of bigtable/gfs you would need freenet. You might even have to give up on locking completely. Then on the presentation side, each node would have to serve web pages using this store, running distributed programs on a safe VM.

Among other lessons, the Google papers show that if you have the platform, the applications come a lot easier. That their subversion code hosting uses Bigtable as a backend is a no-brainer.

Well. Enough incoherent babbling for one morning!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [code not burgers](/archives/2010/04/05/code-not-burgers)
- [the wax wing bits](/archives/2007/11/14/the-wax-wing-bits)
- [\-\\*\- text -\*-](/archives/2007/05/22/text)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [two mechanisms for dynamic type checks](/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)

### One response

1. [Ricky Youngblood](http://www.macgregor26.com/) says:[19 September 2006 11:28 AM](#204)

   Nice. Nothing like an excited dork to elevate my mood a few notches.

Comments are closed.
