---
title: NSF Data Management Plans
url: https://blog.regehr.org/archives/662
published: "2012-01-10T04:12:04Z"
feed: regehr
guid: http://blog.regehr.org/?p=662
---

# NSF Data Management Plans

As of a year ago, all grant proposals submitted to NSF must be accompanied by a [data management plan](http://www.nsf.gov/pubs/policydocs/pappguide/nsf11001/gpg_2.jsp#dmp). Basically, the PIs must explain:

- how sensitive data (for example, data that contains personal information about experimental subjects) will be managed
- how data resulting from the research will be archived and how access to it will be preserved

The requirements seem reasonable and certainly people asking for public funding should have thought about these issues. The problem is that little guidance is given about what constitutes an appropriate plan, or how the quality of the plan will play into the evaluation of the proposal. Since I (happily!) do not work with sensitive data, this post will deal primarily with the second item — preserving public access to data.

The [NSF policy on Dissemination and Sharing of Research Results](http://www.nsf.gov/pubs/policydocs/pappguide/nsf11001/aag_6.jsp#VID4) makes it clear that researchers are expected to share the results of their research in three ways. First, research results must be published. Second, peripheral data must be shared “at no more than incremental cost and within a reasonable time.” Third, NSF grantees are permitted to “retain principal legal rights to intellectual property developed under NSF grants to provide incentives for development and dissemination of inventions, software and publications that can enhance their usefulness, accessibility and upkeep.” In other words, it is permissible to create proprietary products based on technologies developed using NSF money.

It is the second item in the previous paragraph that concerns us here, and which seems to be a main focus of the data management plan. The problems that PIs need to solve when writing this plan include determining:

- How much data will need to be archived? A few megabytes of code or a few petabytes of imaging data?
- Over what time period must data be stored? The duration of the grant or 75 years?
- What kind of failures must be tolerated? The failure of a single disk or tape? A major, region-wide disaster ( [like the one Salt Lake City is certain to get, eventually](http://geology.utah.gov/utahgeo/hazards/eqfault/eqfact.htm))?
- In the absence of disasters, how many bit flips in archived data are permissible per year?

Of course the cost of implementing such a plan might vary over many orders of magnitude depending on the answers. Furthermore, there is the difficult question of how to pay for archiving services after the grant expires. I’m personally a bit cynical about the ability of an educational organization to make a funding commitment that will weather unforeseeable financial problems and changes in administration. Also, I’m guessing that data management plans along the lines of “we’ll mail a couple of TB disks to our buddies across the country” aren’t going to fly anymore.

For large-scale data, a serious implementation of the NSF’s requirements seems to be beyond the capabilities of most individual PIs and probably also out of reach of smallish organizations like academic departments. Rather, it will make sense to pool resources at least across an entire institution and probably across several of them. [Here’s an example](http://www.icpsr.umich.edu/icpsrweb/ICPSR/curation/index.jsp) where this is happening for political and social research. [Here’s some guidance](http://campusguides.lib.utah.edu/content.php?pid=126157&sid=2131664) provided by my institution, which includes the [Uspace](http://uspace.utah.edu/) repository. I don’t believe they are (yet) equipped to handle very large data sets, and I’d guess that similar repositories at other institutions are in the same boat.

This post was motivated by the fact that researchers like me, who haven’t ever had to worry about data management — because we don’t produce much data — now need to care about it, at least to the level of producing a plausible plan. This has lead to a fair amount of confusion among people I’ve talked to. I’d be interested to hear what’s going on at other US CS departments to address this.
