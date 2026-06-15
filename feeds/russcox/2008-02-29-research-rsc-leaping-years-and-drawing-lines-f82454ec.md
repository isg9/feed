---
title: 'research!rsc: Leaping Years and Drawing Lines'
url: https://research.swtch.com/leap
published: "2008-02-29T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/leap
---

# research!rsc: Leaping Years and Drawing Lines

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Leaping Years and Drawing Lines Russ Cox February 29, 2008 *research.swtch.com/leap* Posted on Friday, February 29, 2008.

Some of the earliest mathematics was invented to design calendars. Since the Earth rotates around its axis about 365.25 times per revolution around the sun, some years in a solar calendar need to be 365 days long and others 366 days long in order to keep the calendar in sync with the sun. The key challenge in designing a calendar is to find a pattern of short and long years that achieve the desired approximation. The Julian calendar uses the simple pattern of one leap year every four years. The Islamic calendar, which syncs against the moon, needs to have years approximating 354.37 days per year. To do this, it inserts 11 leap years over a 30 year cycle.

Bresenham's algorithm chooses which pixels to use when drawing a bitmap representation of a line. The basic idea, for a mostly horizontal line that angles upward, is to move left to right, one pixel at a time, moving upward when the distance between the actual y and the pixel-approximate y has grown too large.

```
drawline(dx, dy)    /* mostly horizontal line, upward line */
    x = 0
    y = 0
    error = 0
    while (x < dx || y < dy)
        drawpoint(x, y);
        x++;
        if (x*(dy/dx) - y > 0.5)
            y++;
    drawpoint(x, y);

```

For example, drawing a line 30 pixels long and 12 pixels high produces

![](30x12.png)

In their 2004 paper “ **[Line Drawing,\** **Leap Years, and Euclid](http://emr.cs.uiuc.edu/~reingold/bresenham.pdf)**” (PDF), Mitchell Harris and Edward Reingold show that the problem of choosing a leap year pattern is a generalized version of Bresenham's line drawing problem. This is surprising, but makes intuitive sense when you realize that both are concerned with computing an integer approximation to a rational fraction as evenly as possible. Leap year computations are basically drawing a line with slope 1/365.25 (or 1/364.37): each pixel is a day, and its y coordinate determines which year it belongs to.

Harris and Reingold go on to show that Euclid's algorithm can compute leap year patterns! Euclid's algorithm usually computes the greatest common divisor of two numbers:

```
gcd(u, v)
    while (u != v)
        if (u > v)
            u = u mod v;
        else
            v = v mod u;
    return u;

```

The connection is that if you're trying to draw a line *dx* by *dy* but *dx* and *dy* have some common divisor *d*, then the Bresenham algorithm will simply emit the pattern for a line *dx/d* by *dy/d*, *d* times. The example shown above can be viewed as two 15x6 lines, three 10x4 lines, or six 5x2 lines:

![](euclid.png)

Euclid's algorithm gives the largest possible *d*, and you can adapt the algorithm to build the associated repeating pattern while it computes *d*.

In [Volume 2](http://www-cs-faculty.stanford.edu/~knuth/taocp.html#vol2), Donald Knuth calls it “the granddaddy of all algorithms, because it is the oldest nontrivial algorithm that has survived to the present day.” It also seems to have some famous children.

*Further reading*:

Bresenham described his algorithm in his 1965 paper “ [Algorithm for computer control of a digital plotter](http://www.research.ibm.com/journal/sj/041/ibmsjIVRIC.pdf)”

Godfried Toussaint's 2005 paper “ [The Euclidean algorithm generates traditional musical rhythms](http://cgm.cs.mcgill.ca/~godfried/publications/banff.pdf)” shows yet another pattern connection, this time to rhythms.

The Gregorian calendar is an evolved version of the Julian calendar and can't be viewed as a Bresenham line. However, Jeffrey Shallit's 1994 paper “ [Pierce expansions and rules for the determination of leap years](http://www.emis.de/cgi-bin/zmen/ZMATH/en/zmath.html?first=1&maxdocs=3&type=pdf&an=0823.11043&format=complete)” shows how to compute leap years using [Pierce expansions](http://mathworld.wolfram.com/PierceExpansion.html), and this model can generate the Gregorian calendar too.

Edward Reingold and Nashum Dershowitz are the experts on calendar algorithms. Their pair of papers 1990 [Calendrical Calculations](http://emr.cs.uiuc.edu/~reingold/calendar.ps) and 1993 [Calendrical Calculations, II: Three Historical Calendars](http://emr.cs.uiuc.edu/~reingold/calendar2.ps) gives algorithms (and Lisp code) for essentially any date-related computation you might want to do. They expanded upon these papers in their books [Calendrical Calculations: The Millennium Edition](http://emr.cs.iit.edu/home/reingold/calendar-book/second-edition/index.html) and [Calendrical Tabulations: 1900-2200](http://emr.cs.iit.edu/home/reingold/calendar-book/tables/index.html).

(Comments originally posted via Blogger.)

- [Citizen of the world](http://www.blogger.com/profile/14126495860988156429)(March 16, 2008 3:47 PM) "The Islamic calendar, which syncs against the moon, needs to have years approximating 354.37 days per year. To do this, it inserts 11 leap years over a 30 year cycle."

The Islamic calendar does not insert any leap years. As a consequence the islamic months simply rotate through the solar calendar.

The Jewish and the Hindu calendar which are also lunar calendars do in fact insert leap months to synchronize the lunar calendar with a solar calendar.
