---
title: 'research!rsc: Traveling Passenger Problems'
url: https://research.swtch.com/ita
published: "2008-02-13T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/ita
---

# research!rsc: Traveling Passenger Problems

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Traveling Passenger Problems Russ Cox February 13, 2008 *research.swtch.com/ita* Posted on Wednesday, February 13, 2008.

(Today's late post is brought to you by Continental Airlines and the letters E, W, and R.)

[ITA Software](http://www.itasoftware.com/)'s flagship product, [QPX](http://www.itasoftware.com/solutions/qpx.html), is an airline ticket search engine. [You tell it](http://matrix.itasoftware.com/) where you want to fly and when, and it comes back with possible trips at many different prices. QPX powers Orbitz, CheapTickets, Hotwire, and a half dozen airline company sites.

The airlines want to charge as much as possible for each ticket they sell, but still fill each plane. But the airlines can't just ask possible customers how much money they have to spend, and they also can't explicitly set different prices based on who is buying the ticket. So they attach restrictions to less expensive fares, hoping that business travelers won't accept the restrictions and will pay for the more expensive tickets. The most well-known example is that trips that include a Saturday night are cheaper. There are also changes in ticket prices based on the rest of the trip: a one-way ticket from Boston to Newark might cost [$349](http://matrix.itasoftware.com/cvg/dispatch/bonsaiproxy/p?V=M28288728694672439931&AAC=2008-02-13-15&AAA=30&TBS=30&L=B25964187288926772709&J=35&Q=-1), but a one-way ticket from Boston to Raleigh via the very same Newark flight only [$145](http://matrix.itasoftware.com/cvg/dispatch/bonsaiproxy/p?V=M28742177667546740017&AAC=2008-02-13-15&AAA=30&TBS=30&L=B29124577048372107913&J=73&Q=-1)\*. Of course, if you make a habit of buying multi-leg tickets and skipping the connecting flights, the airlines might blacklist you (or so a travel agent once told me).

These restrictions make QPX's job harder than you might think. In fact, the general case is undecidable!

ITA's Chief Scientist, Carl de Marcken, gives a talk titled “ **[Computational Complexity of Airline Travel](http://www.demarcken.org/carl/papers/ITA-software-travel-complexity/img0.html)**&rdquo (HTML, also [PDF](http://www.demarcken.org/carl/papers/ITA-software-travel-complexity/ITA-software-travel-complexity.pdf)), in which he proves that:

> 1\. Even if the airlines publish only a single fare (with every ticket a single one way PU), and all the airports in a passenger's itinerary are fixed, so that the only remaining choice is what flights to use between the consecutive airports, deciding whether there is a valid ticket is NP-hard.
>
> 2\. Fixing the flights and priceable units, but leaving open the choice of fares to pay for each flight, deciding whether there is a valid ticket is NP-hard.
>
> 3\. Removing bounds on the size of solutions, the general question of whether there is a ticket for a query is EXPSPACE-hard. That is, air travel planning is at least as hard (it might be harder) as deciding whether a computer program that can use space exponentially bigger than the input halts. EXPSPACE-hard problems are (thought to be) much harder than NP-complete problems like the traveling salesman problem, and even much harder than PSPACE-complete problems like playing games optimally. There is no practical hope that computers will ever be able to solve EXPSPACE-hard problems perfectly, even if quantum computing becomes a reality.
>
> 4\. The final result shows that just finding out whether there is a valid solution for a query is actually harder than EXPSPACE-complete: it is unsolvable (undecidable). The question of whether a valid ticket exists can not be solved for all databases and all queries no matter how long a computer thinks. However the full proof of this result is considerably more complex than the EXPSPACE-hard proof without offering any greater understanding of the problem.

The NP-hardness proofs reduce 3-color, the EXPSPACE-completeness proof encodes a Turing machine in ticket restrictions, and the undecidability proof encodes Diophantine equations. Here's a [3x3-bit binary multiplier](http://www.demarcken.org/carl/papers/ITA-software-travel-complexity/img43.html). (Be sure to read the early slides too, which give necessary details about ticket pricing.)

\\* As of February 13, Continental flights on March 12, 2008 had those prices. Actual fares may no longer match.
