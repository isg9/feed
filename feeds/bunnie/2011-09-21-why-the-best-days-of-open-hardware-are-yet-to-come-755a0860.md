---
title: Why the Best Days of Open Hardware are Yet to Come
url: https://www.bunniestudios.com/blog/2011/why-the-best-days-of-open-hardware-are-yet-to-come/
published: "2011-09-21T19:11:11Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1863
---

# Why the Best Days of Open Hardware are Yet to Come

Recently, I gave a talk at the [2011 Open Hardware Summit](http://www.openhardwaresummit.org/schedule-2011-2/). The program committee had requested that I prepare a “vision” talk, something that addresses open hardware issues 20-30 years out. These kinds of talks are notoriously difficult to get right, and I don’t really consider myself a vision guy; but I gave it my best shot. Fortunately, the talk was well-received, so I’m sharing the ideas here on my blog.

**Abstract**

Currently, open hardware is a niche industry. In this post, I highlight the trends that have caused the hardware industry to favor large, closed businesses at the expense of small or individual innovators. However, looking 20-30 years into the future, I see a fundamental shift in trends that can tilt the balance of power to favor innovation over scale.

**Where we Came From: Open to Closed**

[![](http://bunniestudios.com/blog/images/ohs11_radio_sm.png)](http://bunniestudios.com/blog/images/ohs11_radio.png)

In the beginning, hardware was open. Early consumer electronic products, such as vacuum tube radios, often shipped with user manuals that contained full schematics, a list of replacement parts, and instructions for service. In the 80’s, computers often shipped with schematics. For example, the Apple II shipped with a reference manual that included a full schematic of the mainboard, an artifact that I credit as strongly influencing me to get into hardware. However, contemporary user manuals lack such depth of information; the most complex diagram in a recent Mac Pro user instructs on how to sit at the computer: “thighs slightly lifted”, “shoulders relaxed”, etc.

![](http://bunniestudios.com/blog/images/ohs11_sitting_diagram.png)

What happened? Did electronics just get too hard and complex? On the contrary, improving electronics got to *easy*: the pace of Moore’s Law has been too much for small-scale innovators to keep up.

**Where we Are: Sit and Wait > Innovate**

Consider this snapshot of Moore’s law, illustrating “goodness” (pick virtually any metric — performance, density, price-per-quanta) doubling every 18 months. This chart is unusual in that the vertical axis is linear. Most charts depicting Moore’s law use a logarithic vertical scale, which flattens the curve’s sharp upward trend into a much more innocuous looking straight line.

![](http://bunniestudios.com/blog/images/ohs11_moore_overlay_orig.png)

*Above: Moore’s Law, doubling once every 18 months (red) versus linear improvement of 75% per year (blue). The small green sliver between the red and blue lines (found at t < 2yrs) represents the window of opportunity where linear improvement exceeds Moore's law. Note that the vertical axis on this graph is linear scale.*

Plotted in blue is a line that represents a linear improvement over time. This might be representative of a small innovator working at a constant but respectable rate of 75% per year, non compounding, to add or improve features on a given platform. The tiny (almost invisible) green sliver between the curves represents the market opportunity of the small innovator versus Moore’s law.

The juxtaposition of these two curves highlights the central challenge facing small innovators over the past three decades. It has been more rewarding to “sit and wait” rather than innovate: if it takes two years to implement an innovation that doubles the performance of a system, one is better served by not trying and simply waiting and upgrading to the latest hardware two years later. It is a Sisyphean exercise to race against Moore’s Law.

This exponential growth mechanic favors large businesses with the resources to achieve huge scale. Instead of developing one product at a time, a competitive business must have the resources and hopefully the vision to develop 3 or 4 generations of products simultaneously. Furthermore, reaching the global market within the timespan of a single technology generation requires a supply chain and distribution channel that can do millions of units a month: consider that selling at a rate of 10,000 units per month would take 8 years to reach “only” a million users, or about 1% of the households in the US alone. And, significantly, the small barrier (a few months time) created by closing a design and forcing the competition to reverse engineer products can be an advantage, especially when contrasted to the pace of Moore’s law. Thus, technology markets have become progressively inaccessible to small innovators over the past three decades as individuals struggle to keep up with the technology treadmill, and big companies continue to close their designs to gain a thin edge on their competition.

However, this trend is changing.

**Where we are Going: Heirloom Laptops**

Below is a plot of Intel CPU clock speed at introduction versus time. There is an abrupt plateau in 2003 where clock speed stopped increasing. Since then, CPU makers have been using multi-core technology to drive performance (effective performance extrapolated as the pink dashed line), but this wasn’t by choice: certain physical limits were reached that prevented practical clock scaling (primarily related to power and wire delay scaling). Transistor density (and hence core count) continues to scale, but the pace is decelerating. In the 90’s, transistor count was doubling once every 18 months; today, it is probably slower than once every 24 months. Soon, transistor density scaling will slow to a pace of 36 months per generation, and eventually it will come to an effective stand-still. The absolute endpoint for transistor scaling is a topic of debate, but [one study](http://www.iwailab.ep.titech.ac.jp/pdf/iwaironbun/0906infos.pdf) indicates that scaling may stop at around 5nm effective gate length sometime around 2020 or 2030 (H. Iwai, Microelectronics Engineering (2009), doi: 10.1016 / j.mee.2009.03.129). 5 nm is about the space between 10 silicon atoms, so even if this guess is wrong, it can’t be wrong by much.

![](http://bunniestudios.com/blog/images/ohs11_clockscaling.png)

The implications are profound. Someday, you cannot rely on buying a faster computer next year. Your phone won’t get any smaller or more powerful. And the flash drive you buy next year will cost the same yet store the same number of bits. The idea of an “heirloom laptop” may sound preposterous today, but someday we may perceive our computers as cherished and useful looms to hand down to our children as part of our legacy.

This slowing trend is good for small businesses, and likewise open hardware practices. To see why this is the case, let’s revisit the plot of Moore’s Law versus linear improvement, but this time overlay two new scenarios: technology doubling once every 24 and 36 months.

![](http://bunniestudios.com/blog/images/ohs11_moore_overlay_log.png)

*Above: Moore’s Law versus linear improvement (blue), plotted against the scenarios of doubling once every 18 months (red), every 24 months (black), and every 36 months (pink). The green area between the pink and blue lines represents the window of opportunity where linear improvement exceeds Moore’s law when the doubling interval is set to 36 months. Note that the vertical axis on this graph is log scale.*

Again, the green area represents the market opportunity for linear improvement vs. Moore’s Law. In the 36-month scenario, linear improvement not only has over 8 years to go before it is lapped by Moore’s Law, there is a point at around year 2 or 3 where the optimized solution is clearly superior to Moore’s Law. In other words, there is a genuine market window for monetizing innovative solutions at a pace that small businesses can handle.

Also, as Moore’s law decelerates, there is a potential for greater standardization of platforms. While today it seems ridiculous to create a standard tablet or mobile phone chassis with interchangeable components, this becomes a reasonable proposition when components stop shrinking and changing so much. As technology decelerates, there will be a convergence between that which is found in mobile phones, and that which is found in embedded CPU modules (such as the Arduino). The creation of stable, performance-competitive open platforms will be enabling for small businesses. Of course, a small business can still choose to be closed, but by doing so it must create a vertical set of proprietary infrastructure, and the dilution of focus to implement such a stack could be disadvantageous.

In the post-Moore’s law future, FPGAs may find themselves performing respectably to their hard-wired CPU kin, for at least two reasons: the flexible yet regular structure of an FPGA may lend it a longer scaling curve, in part due to the FPGA’s ability to reconfigure circuits around small-scale fluctuations in fabrication tolerances, and because the extra effort to optimize code for hardware acceleration will amortize more favorably as CPU performance scaling increasingly relies upon difficult techniques such as massive parallelism. After all, today’s massively multicore CPU architectures are starting to look a lot like the coarse-grain FPGA architectures proposed in academic circles in the mid to late 90’s. An equalization of FPGA to CPU performance should greatly facilitate the penetration of open hardware at a very deep level.

There will be a rise in repair culture as technology becomes less disposable and more permanent. Replacing worn out computer parts five years from their purchase date won’t seem so silly when the replacement part has virtually the same specifications and price as the old part. This rise in repair culture will create a demand for schematics and spare parts that in turn facilitates the growth of open ecosystems and small businesses.

Personally, I’m looking forward to the return of artisan engineering, where elegance, optimization and balance are valued over feature creep, and where I can use the same tool for a decade and not be viewed as an anachronism (most people laugh when they hear my email client is still Eudora 7).

**Examples**

The deceleration of Moore’s Law is already showing its impact on markets that are not as sensitive to performance. Consider the rise of the Arduino platform. The Arduino took several years to gain popularity, with virtually the same hardware at its core since 2005. Fortunately, the demands of Arduino’s primary market (physical computing, education, and embedded control applications) has not grown and thus the platform can be very stable. This stability in turn has enabled Arduino to grow deep roots in a thriving user community with open and interoperable standards.

Another example is the [Shanzhai phenomenon](http://www.bunniestudios.com/blog/?p=284) in China; in a nutshell, the Shanzhai are typically small businesses, and they rely upon an ecosystem of “shared” blueprints. The Shanzhai are masters at building low-end “feature phones”. The market for feature phones is largely insensitive to improvements in CPU technology; you don’t need a GHz-class CPU to drive the simple UIs found on feature phones. Thus, the same core chipset can be re-used for years with little adverse impact on demand or competitiveness of the final product. This platform stability has afforded these small, agile innovators the time to learn the platform thoroughly, and to recover this investment by creating riff after riff on the same theme. Often times, the results are astonishingly innovative, yet accomplished on a shoe string budget. Initially, the Shanzhai were viewed simply as copycatters; but thanks to the relative stability of the feature phone platform, they have learned their tools well and are now pumping out novel and creative works.

The scene is set for the open hardware ecosystem to blossom over the next couple of decades, with some hard work and a bit of luck. The inevitable slowdown of Moore’s Law may spell trouble for today’s technology giants, but it also creates an opportunity for the fledgling open hardware movement to grow roots and be the start of something potentially very big. In order to seize this opportunity, today’s open hardware pioneers will need to set the stage by creating a culture of permissive standards and customs that can scale into the future.

I look forward to being a part of open hardware’s bright future.
